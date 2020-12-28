---
title: Kafka——Rebalance过程
comments: true
tags: 
- Kafka
- BigData
categories: 
- technology
keywords: Kafka,Rebalance,过程
toc: true
date: 2020-07-08 15:35:05
---


> Rebalance（重平衡 ）本质上是一种协议， 规定了一个Consumer Group下的所有 Consumer 如何达成一致， 来分配订阅Topic的每个分区。 说简单点就是 给消费组每个消费者分配消费任务的过程。

### 触发

- 订阅信息变化(partition变化，topic变化)
- ConsumerGroup组内成员变化 (心跳超时/Consumer 加入/Consumer退出)

### **过程**

1. FindCoordinator - 寻找管理当前Group的GroupCoordinator的Node信息
2. JoinGroup - 向GroupCoordinator发送加入信息
3. SyncGroup - Group Leader 上传分区信息到Coordinator，Coordinator下发分区信息到每个Consumer

**Server**                           **Client**                      

- FindCoordinator

    ←  FindCoordinatorRequest  

    `key` - `groupId` 

    `keyType` - `Group`

    向负载最小的Broker节点发送请求

    ```java
    public Node leastLoadedNode(long now) {
            List<Node> nodes = this.metadataUpdater.fetchNodes();
            if (nodes.isEmpty())
                throw new IllegalStateException("There are no nodes in the Kafka cluster");
            int inflight = Integer.MAX_VALUE;

            Node foundConnecting = null;
            Node foundCanConnect = null;
            Node foundReady = null;

            int offset = this.randOffset.nextInt(nodes.size());
            for (int i = 0; i < nodes.size(); i++) {
                int idx = (offset + i) % nodes.size();
                Node node = nodes.get(idx);
                if (canSendRequest(node.idString(), now)) {
                    int currInflight = this.inFlightRequests.count(node.idString());
                    if (currInflight == 0) {
                        // if we find an established connection with no in-flight requests we can stop right away
                        log.trace("Found least loaded node {} connected with no in-flight requests", node);
                        return node;
                    } else if (currInflight < inflight) {
                        // otherwise if this is the best we have found so far, record that
                        inflight = currInflight;
                        foundReady = node;
                    }
                } else if (connectionStates.isPreparingConnection(node.idString())) {
                    foundConnecting = node;
                } else if (canConnect(node, now)) {
                    if (foundCanConnect == null ||
                            this.connectionStates.lastConnectAttemptMs(foundCanConnect.idString()) >
                                    this.connectionStates.lastConnectAttemptMs(node.idString())) {
                        foundCanConnect = node;
                    }
                } else {
                    log.trace("Removing node {} from least loaded node selection since it is neither ready " +
                            "for sending or connecting", node);
                }
            }

            // We prefer established connections if possible. Otherwise, we will wait for connections
            // which are being established before connecting to new nodes.
            if (foundReady != null) {
                log.trace("Found least loaded node {} with {} inflight requests", foundReady, inflight);
                return foundReady;
            } else if (foundConnecting != null) {
                log.trace("Found least loaded connecting node {}", foundConnecting);
                return foundConnecting;
            } else if (foundCanConnect != null) {
                log.trace("Found least loaded node {} with no active connection", foundCanConnect);
                return foundCanConnect;
            } else {
                log.trace("Least loaded node selection failed to find an available node");
                return null;
            }
        }
    ```

    →  FindCoordinatorResponse  

    `host` `port` `nodeId`

    Hash( `groupId` ) % `__consumer_offsets` Topic `partitions` 的 `leader` 节点

- JoinGroup

    ←   JoinGroupRequest    向Coordinator发送

    `groupId`

    `memberId` - 

    `groupInstanceId` - 静态ID

    `protocols` - `assignors`

    `isLeader` = `false`

    清空`subscriptions` 中的topic信息

    →   JoinGroupResponse

    `leader`

    `memberId`

    `members` - 成员信息，只有Leader才会有值，Follower是空列表

    第一个加入的 `memeberId` 成为Leader

    Coordinator会等待一段时间，取决于Consumer的`max.poll.interval.ms`

    Group状态变为  `PreparingRebalance` ，Request会被阻塞，直到所有的member都发来JoinGroupRequest后，执行回调并修改Group状态为`CompletingRebalance`

- SyncGroup

    ← SyncGroupRequest

    `groupId` 

    `generationId` - 年代信息

    `memberId`

    `groupInstanceId`

    `assignments`  -   `memberId - topicPartitions`

    Leader进行分区，并将分区结果发送给Coordinator

    → SyncGroupResponse

    `assignment` - 分区结果

    Leader的分区结果没有到的时候，Group状态为`CompletingRebalance`，SyncGroupRequest被阻塞，直到Leader的分区结果到了之后,Group状态变为 `Stable`,执行回调下发分区信息

    之后Consumer根据传回来的分区结果去更新自己的订阅信息

### Heartbeat

心跳机制，主要用于确认双方是否存活，以及Group状态信息

Server

← HeartbeatRequest

`groupId`
`generationId`
`memberId`
`groupInstanceId`

→ HeartbeatResponse

`errorCode`

**Server**

```scala
group.currentState match {
  case Empty =>
    responseCallback(Errors.UNKNOWN_MEMBER_ID)

  case CompletingRebalance =>
      responseCallback(Errors.REBALANCE_IN_PROGRESS)

  case PreparingRebalance =>
      val member = group.get(memberId)
      completeAndScheduleNextHeartbeatExpiration(group, member)
      responseCallback(Errors.REBALANCE_IN_PROGRESS)

  case Stable =>
      val member = group.get(memberId)
      completeAndScheduleNextHeartbeatExpiration(group, member)
      responseCallback(Errors.NONE)

  case Dead =>
    throw new IllegalStateException(s"Reached unexpected condition for Dead group $groupId")
}
```

如果Group状态为 `Stable` error为 `None` ，为其他，则有错误码

如果心跳超时，则会更新`group` `member`信息，踢掉超时的`member` ，并修改Group状态为`PreparingRebalance`

```scala
def onExpireHeartbeat(group: GroupMetadata, memberId: String, isPending: Boolean): Unit = {
  group.inLock {
    if (group.is(Dead)) {
      info(s"Received notification of heartbeat expiration for member $memberId after group ${group.groupId} had already been unloaded or deleted.")
    } else if (isPending) {
      info(s"Pending member $memberId in group ${group.groupId} has been removed after session timeout expiration.")
      removePendingMemberAndUpdateGroup(group, memberId)
    } else if (!group.has(memberId)) {
      debug(s"Member $memberId has already been removed from the group.")
    } else {
      val member = group.get(memberId)
      if (!member.hasSatisfiedHeartbeat) {
        info(s"Member ${member.memberId} in group ${group.groupId} has failed, removing it from the group")
        removeMemberAndUpdateGroup(group, member, s"removing member ${member.memberId} on heartbeat expiration")
      }
    }
  }
}
```

```scala
private def removeMemberAndUpdateGroup(group: GroupMetadata, member: MemberMetadata, reason: String): Unit = {
  // New members may timeout with a pending JoinGroup while the group is still rebalancing, so we have
  // to invoke the callback before removing the member. We return UNKNOWN_MEMBER_ID so that the consumer
  // will retry the JoinGroup request if is still active.
  group.maybeInvokeJoinCallback(member, JoinGroupResult(JoinGroupRequest.UNKNOWN_MEMBER_ID, Errors.UNKNOWN_MEMBER_ID))

  group.remove(member.memberId)
  group.removeStaticMember(member.groupInstanceId)

  group.currentState match {
    case Dead | Empty =>
    case Stable | CompletingRebalance => maybePrepareRebalance(group, reason)
    case PreparingRebalance => joinPurgatory.checkAndComplete(GroupKey(group.groupId))
  }
}
```

**Consumer**

Consumer收到返回结果后，会查看是否有错误信息，如果收到正在Rebalance的错误，就会将ReJoin的标志位置为`True`

```java
public void handle(HeartbeatResponse heartbeatResponse, RequestFuture<Void> future) {
    sensors.heartbeatSensor.record(response.requestLatencyMs());
    Errors error = heartbeatResponse.error();
    if (error == Errors.NONE) {
        log.debug("Received successful Heartbeat response");
        future.complete(null);
    } else if (error == Errors.COORDINATOR_NOT_AVAILABLE
            || error == Errors.NOT_COORDINATOR) {
        log.info("Attempt to heartbeat failed since coordinator {} is either not started or not valid",
                coordinator());
        markCoordinatorUnknown();
        future.raise(error);
    } else if (error == Errors.REBALANCE_IN_PROGRESS) {
        log.info("Attempt to heartbeat failed since group is rebalancing");
        requestRejoin();
        future.raise(error);
    } else if (error == Errors.ILLEGAL_GENERATION ||
               error == Errors.UNKNOWN_MEMBER_ID ||
               error == Errors.FENCED_INSTANCE_ID) {
        if (generationUnchanged()) {
            log.info("Attempt to heartbeat with {} and group instance id {} failed due to {}, resetting generation",
                sentGeneration, rebalanceConfig.groupInstanceId, error);
            resetGenerationOnResponseError(ApiKeys.HEARTBEAT, error);
            future.raise(error);
        } else {
            // if the generation has changed, then ignore this error
            log.info("Attempt to heartbeat with stale {} and group instance id {} failed due to {}, ignoring the error",
                sentGeneration, rebalanceConfig.groupInstanceId, error);
            future.complete(null);
        }
    } else if (error == Errors.GROUP_AUTHORIZATION_FAILED) {
        future.raise(GroupAuthorizationException.forGroupId(rebalanceConfig.groupId));
    } else {
        future.raise(new KafkaException("Unexpected error in heartbeat response: " + error.message()));
    }
}
```

```java
public synchronized void requestRejoin() {
    this.rejoinNeeded = true;
}
```

### Server Request Handle

```java
case ApiKeys.FIND_COORDINATOR => handleFindCoordinatorRequest(request)
case ApiKeys.JOIN_GROUP => handleJoinGroupRequest(request)
case ApiKeys.HEARTBEAT => handleHeartbeatRequest(request)
case ApiKeys.LEAVE_GROUP => handleLeaveGroupRequest(request)
case ApiKeys.SYNC_GROUP => handleSyncGroupRequest(request)
case ApiKeys.DESCRIBE_GROUPS => handleDescribeGroupRequest(request)
```

### Server Group State

`PreparingRebalance` Group is preparing to rebalance

`CompletingRebalance` Group is awaiting state assignment from the leader

`Stable` Group is stable

`Dead` Group has no more members and its metadata is being removed

`Empty` Group has no more members, but lingers until all offsets have expired.

### Client Member State

`UNJOINED`     the client is not part of a group
`REBALANCING` the client has begun rebalancing
`STABLE` the client has joined and is sending heartbeats
