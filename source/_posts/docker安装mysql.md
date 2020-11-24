---
title: docker安装mysql
tags: 
- MySQL
- Docker
comments: true
categories: 
  - technology
originContent: |-
  使用Docker安装MySQL8.0，并修改字符集以及用户授权
toc: false
keywords: Docker,MySQL,8.0,字符集
date: 2020-02-20 08:40:51
---
使用Docker安装MySQL8.0，并修改字符集以及用户授权

### 拉取镜像
```
docker pull mysql:8.0
```

### 运行容器
`-p` 映射本地端口
`MYSQL_ROOT_PASSWORD` root密码
`-v` 挂载本地卷

```
docker run \
-p 3306:3306 \
--name mysql \
-v ~/top/data/mysql/logs:/var/log/mysql \
-v ~/top/data/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=admin \
-d mysql:8.0 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

### 修改字符集
- 进入容器
```
docker exec -it mysql bash
```
- 更换国内源,以`Debian`-`buster`为例
```
echo "deb http://mirrors.aliyun.com/debian/ buster main non-free contrib" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/debian-security buster/updates main" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/debian-security buster/updates main" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib" >> /etc/apt/sources.list
```

- 安装`vim`
```
apt-get update && apt-get install vim -y
```
- 修改MySQL字符集  
```
vim /etc/mysql/conf.d/mysql.cnf
```
增加如下内容保存退出
  
```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```
- 链接MySQL  
```
mysql -uroot -padmin
```
- 确认编码
```
mysql> show variables like'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8                           |
| character_set_connection | utf8                           |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8                           |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8                           |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.01 sec)
```

### 授权外部访问
```
grant all privileges on *.* to 'root'@'%';
```