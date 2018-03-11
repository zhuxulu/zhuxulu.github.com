---
layout: post
title: docker 持续集成（4） - 安装 tomcat
categories:
- dev
tags:
- ci
- docker
- jenkins
- gitlab
- java
- maven
- tomcat
---

#### 安装 tomcat

执行以下命令安装 tomcat。

    docker pull tomcat:7.0.85-jre7

执行以下命令运行 tomcat。

    docker run -itd --rm -p 8888:8080 tomcat:7.0.85-jre7

或者，将宿主机的文件或目录映射到容器中，并运行 tomcat：

    docker run -itd --rm -p 8888:8080 \
        -v $PWD/target/easygo.war:/usr/local/tomcat/webapps/easygo.war \
        --name tomcat \
        tomcat:7.0.85-jre7

参考：[https://hub.docker.com/_/tomcat/](https://hub.docker.com/_/tomcat/)

