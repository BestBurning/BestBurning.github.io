---
title: docker-compost安装单机Elasticsearch
tags: 
- Elasticsearch
- Docker
comments: true
categories: 
  - technology
toc: false
keywords: Docker,Elasticsearch,Kibana
date: 2020-11-20 08:40:51
---

使用`docker-compose`安装`Elasticsearch` `7.9.3`


### 确认`docker-compose`已经安装

```
➜  elasticsearch-docker docker-compose version


docker-compose version 1.27.4, build 40524192
docker-py version: 4.3.1
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.1g  21 Apr 2020

```
### docker-compose.yml
```
version: '2'
services:

  elasticsearch:
    image: elasticsearch:7.9.3
    container_name: elasticsearch7.9.3
    environment:
      - bootstrap.memory_lock=true   # 内存交换的选项，官网建议为true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m" # 设置内存，如内存不足，可以尝试调低点
      - discovery.type=single-node      # 是否启用单节点模式
    ulimits:        # 栈内存的上限
      memlock:
        soft: -1    # 不限制
        hard: -1    # 不限制
    # volumes:
    #  - ~/top/data/elasticsearch/config:/usr/share/elasticsearch/config
    #  - ~/top/data/elasticsearch/data:/usr/share/elasticsearch/data
    #  - ~/top/data/elasticsearch/logs:/usr/share/elasticsearch/logs
    hostname: elasticsearch
    restart: always
    ports:
      - 9200:9200
      - 9300:9300


  kibana:
    image: kibana:7.9.3
    container_name: kibana7.9.3
    environment:
      - elasticsearch.hosts=http://elasticsearch:9200
    hostname: kibana
    depends_on:
      - elasticsearch
    restart: always
    ports:
      - 5601:5601

volumes:
  esdata:
    driver: local
```

### 运行

```
docker-compose up -d
```

### 将容器内的`config`配置文件复制到需要挂载的目录
```
docker cp -r elasticsearch7.9.3:/usr/share/elasticsearch/config ~/top/data/elasticsearch/

```

### 删除刚刚创建好的容器

```
docker rm elasticsearch7.9.3 kibana7.9.3
```

### 打开`docker-compose.yml`中挂载目录的注释

```
    volumes:
      - ~/top/data/elasticsearch/config:/usr/share/elasticsearch/config
      - ~/top/data/elasticsearch/data:/usr/share/elasticsearch/data
      - ~/top/data/elasticsearch/logs:/usr/share/elasticsearch/logs
```

### 运行

```
docker-compose up -d
```

### 运行成功

Elasticsearch: 访问`9200`端口
Kibana: 访问`5601`端口
