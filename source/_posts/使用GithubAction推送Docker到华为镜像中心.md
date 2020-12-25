---
title: 使用GithubAction推送Docker到香港华为镜像中心
tags: 
- Docker
- Github
comments: true
categories: 
  - technology
toc: true
keywords: Docker,Github Action,华为镜像中心
date: 2020-11-21 08:40:51
---

这次的`CI`目标是使用`GithubAction`构建Docker并推至自己的服务器运行

具体过程如下
`Java`项目`Push`新标签的时候，触发`Github Action`构建`jar`与`Docker`，并推送至[香港华为镜像中心](https://www.huaweicloud.com/product/swr.html?fromacct=cb4bb004-b347-4a92-97af-77d47099a091&utm_source=V1g3MDY4NTY=&utm_medium=cps&utm_campaign=201905)，之后从自己服务器`Pull`并运行。
![](https://images.di1shuai.com/Fowol7SghHczOY-Xx8mrjS90fRWN)

那么为什么要使用[香港华为镜像中心](https://www.huaweicloud.com/product/swr.html?fromacct=cb4bb004-b347-4a92-97af-77d47099a091&utm_source=V1g3MDY4NTY=&utm_medium=cps&utm_campaign=201905)呢？

![](https://images.di1shuai.com/Fiba8IkZK09g3ERJCRW1FP34YmR_)

当然是因为 ~~它免费~~ **穷**啊，
另一个原因就是使用香港的镜像中心**速度**体验非常好，从`GithubAction`推一个100多M的镜像只用`34s`

![](https://images.di1shuai.com/FkVj-c1gISJspivwkbSoXeTHuSFW)

### `Dockerfile`配置

注意`Dockerfile`文件的路径与`Pom`平级

```Dockerfile
FROM openjdk:11-jre
MAINTAINER Shea<zhushuai026@gmail.com>

VOLUME /tmp
EXPOSE 8080

ARG JAR_FILE
ADD target/${JAR_FILE} /root/app.jar
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar /root/app.jar"]
```

### `Pom`配置

使用`dockerfile-maven-plugin`插件可以直接`mvn package`打出`Docker Image`

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 指定该Main Class为全局的唯一入口 -->
                    <mainClass>com.di1shuai.questionbank.CmbQuestionBankApiApplication</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!--可以把依赖的包都打包到生成的Jar包中-->
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.13</version>
                <executions>
                    <execution>
                        <id>default</id>
                        <goals>
                            <goal>build</goal>
                            <goal>push</goal>
                            <goal>tag</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <repository>${project.artifactId}</repository>
                    <tag>${project.version}</tag>
                    <buildArgs>
                        <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### `Github Repository`配置

在`Seetings` -> `Secrets`进行配置

```properties
DOCKER_ORGANIZTION  # 华为镜像中的组织 e.g. di1shuai
DOCKER_PASSWORD     # 华为云AK与SK
DOCKER_REGION       # 所属区域 e.g. ap-southeast-1
DOCKER_REGISTRY     # 镜像中心 e.g. swr.ap-southeast-1.myhuaweicloud.com
DOCKER_USERNAME     # 华为云AK
HOST                # 部署服务器Host e.g. 111.111.111.111 or xxxxx.di1shuai.com
PORT                # 部署服务器端口 e.g. 22
PWD                 # 部署服务器密码 e.g. HelloWorld123
USER                # 部署服务器账号 e.g. tom
```

其中`DOCKER_USERNAME`与`DOCKER_PASSWORD`的值需要去[华为云官网](https://www.huaweicloud.com/product/swr.html?fromacct=cb4bb004-b347-4a92-97af-77d47099a091&utm_source=V1g3MDY4NTY=&utm_medium=cps&utm_campaign=201905)

账号 -> 我的凭证 -> 访问秘钥 -> 新增访问秘钥 -> 获得`AK`和`SK`

1. `DOCKER_USERNAME`=`AK`

2. `DOCKER_PASSWORD`需要在`Linux/Macos`机器上运行

```bash
printf "$AK" | openssl dgst -binary -sha256 -hmac "$SK" | od -An -vtx1 | sed 's/[ \n]//g' | sed 'N;s/\n//'
```

将`$AK`与`$SK`替换为刚刚拿到的`AK`和`SK`

### `Action`配置

路径： `.github/workflows/maven.yml`

```yml
name: Java Deploy with Maven

on:
  create:
    tags:
      - v*
    branches: [master]

jobs:
  build:
    env:
      DOCKER_PROJECT: cmb-question-bank-api 
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Cache Maven packages
        uses: actions/cache@v2
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build with Maven
        run: |
          mvn package -Dmaven.test.skip=true
      - name: Init Env
        run: |
          echo "DOCKER_VERSION=`docker images | grep $DOCKER_PROJECT | awk 'NR==1 {print $2}'`" >> $GITHUB_ENV
          echo "DOCKER_IMAGE_NAME=${{secrets.DOCKER_REGISTRY}}/${{secrets.DOCKER_ORGANIZTION}}/$DOCKER_PROJECT" >> $GITHUB_ENV
      - name: Push Docker
        run: |
          docker login -u ${{secrets.DOCKER_REGION}}@${{secrets.DOCKER_USERNAME}} -p ${{secrets.DOCKER_PASSWORD}} ${{secrets.DOCKER_REGISTRY}}
          docker images
          docker tag $DOCKER_PROJECT:$DOCKER_VERSION $DOCKER_IMAGE_NAME:$DOCKER_VERSION
          docker tag $DOCKER_IMAGE_NAME:$DOCKER_VERSION $DOCKER_IMAGE_NAME:latest
          docker images
          docker push $DOCKER_IMAGE_NAME:$DOCKER_VERSION
          docker push $DOCKER_IMAGE_NAME:latest

  pull-docker:
    env:
      DOCKER_PROJECT: cmb-question-bank-api   
    needs: [build]
    name: Pull Docker
    runs-on: ubuntu-latest
    steps:
      - name: Init Env 
        run: |
          echo "DOCKER_VERSION=`docker images | grep $DOCKER_PROJECT | awk 'NR==1 {print $2}'`" >> $GITHUB_ENV
          echo "DOCKER_IMAGE_NAME=${{secrets.DOCKER_REGISTRY}}/${{secrets.DOCKER_ORGANIZTION}}/$DOCKER_PROJECT" >> $GITHUB_ENV
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          password: ${{ secrets.PWD }}
          port: ${{ secrets.PORT }}
          envs: DOCKER_IMAGE_NAME,DOCKER_PROJECT
          script: |
            docker stop $(docker ps --filter ancestor=$DOCKER_IMAGE_NAME -q)
            docker rm -f $(docker ps -a --filter ancestor=$DOCKER_IMAGE_NAME:latest -q)
            docker rmi -f $(docker images  $DOCKER_IMAGE_NAME:latest -q)
            docker pull $DOCKER_IMAGE_NAME:latest
            cd /opt/apps/
            docker-compose up -d
```

其中`DOCKER_PROJECT: cmb-question-bank-api`替换为你自己的项目名

### 推送一个新标签

`Git`推送一个`v1.0.0`到Github

![](https://images.di1shuai.com/FtBphMALiRu8H1oivWQsIHdiW819)

整个构建过程大约花费`两分半`,还不要钱！
