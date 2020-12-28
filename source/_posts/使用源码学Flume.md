---
title: 使用源码学Flume
comments: true
date: 2017-1-13
keywords: flume,源码
tags: 
- Flume
- BigData
categories: 
- technology

---
 使用flume源码进行学习的话会比较有直观的感受

### flume资源
-	flume官网：		http://flume.apache.org/
-	flume源码：		https://github.com/apache/flume
-	flume最新文档：	http://flume.apache.org/FlumeUserGuide.html#		英文原版，看不懂可以使用google翻译后对比着看


# 使用源码进行flume学习

1. 使用git下载好flume源码
2. 切换到  flume-1.7 分支
3. 打开DEVNOTES文件，这个是开发者需要知道的一些东西，可以看到
 
 ```

=== Running the most recent build

To run the most recent build of Flume, first build the distribuion
packages.

----
mvn install -DskipTests
----

 ```

 使用mvn install -DskilTests可以构建编译flume源码
 过程中可能遇到
 ```
ERROR] Failed to execute goal on project flume-ng-morphline-solr-sink:
Could not resolve dependencies forproject
org.apache.flume.flume-ng-sinks:flume-ng-morphline-solr-sink:jar:1.5.0:
Failed to collect dependencies for[org.apache.flume:flume-ng-core:jar:1.5.0
(compile), org.slf4j:slf4j-api:jar:1.6.1(compile),
org.kitesdk:kite-morphlines-all:pom:0.12.0(compile?),
org.slf4j:jcl-over-slf4j:jar:1.6.1(provided),
org.apache.solr:solr-test-framework:jar:4.3.0(test),
org.kitesdk:kite-morphlines-solr-core:jar:tests:0.12.0(test),
junit:junit:jar:4.10(test)]: Failed to read artifact descriptor
forua_parser:ua-parser:jar:1.3.0: Could not transfer artifact
ua_parser:ua-parser:pom:1.3.0from/to maven-twttr (http://maven.twttr.com):
Connection to http://maven.twttr.com refused: Connection timed out -> [Help 1
```
的问题
在pom中添加一个仓库
```
<repository>
  <id>nexus.axiomalaska.com</id>
  <url>http://nexus.axiomalaska.com/nexus/content/repositories/public</url>
</repository>
```
在添加过程中，我自己少加了
```
<repositories>...</repositories>
```
所以顺便贴上pom全部的结构：
```

 <project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
  <parent>...</parent>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>
 
  <!-- Build Settings -->
  <build>...</build>
  <reporting>...</reporting>
 
  <!-- More Project Information -->
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>
 
  <!-- Environment Settings -->
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
```




