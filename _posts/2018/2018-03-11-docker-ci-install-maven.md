---
layout: post
title: docker 持续集成（3） - 安装 maven
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

#### 安装 maven

执行以下命令安装 maven

    docker pull maven

使用方法：

    docker run -it --rm --name my-maven-project \
        -v "$(pwd)":/project \
        -v /usr/tmp/.m2:/root/.m2 \
        -w /project \
        maven:3.5.3-jdk-8 \
        mvn package

* docker run，创建了 maven:3.5.3-jdk-8 镜像的一个实例。该实例中执行了 mvn package 命令。原则上，这不会影响主机系统。
* -v $(pwd):/project，将当前目录挂载到容器中，作为 /project 目录。这样以来，容器就可以读写主机系统的当前目录了。
* -v /usr/tmp/.m2:/root/.m2，使用卷缓存 Maven 本地仓库。
* -w /project，设置了 /project 作为工作目录。这意味着执行 mvn 命令将在 project 目录中有效。
* --rm，将在执行完毕后删除容器。

这与在主机上直接运行 mvn package 的结果是一样的，只是不必实际安装 Java或 Maven。

可以运行maven clean命令清理项目：

    docker run -it --rm \
        -v "$(pwd)":/project \
        -v /usr/tmp/.m2:/root/.m2 \
        -w /project \
        maven:3.5.3-jdk-8 \
        mvn clean


参考：[http://www.infoq.com/cn/articles/docker-executable-images](http://www.infoq.com/cn/articles/docker-executable-images)
