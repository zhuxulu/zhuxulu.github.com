---
layout: post
title: docker 持续集成（1） - 安装 jenkins
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

#### 安装 jenkins

执行以下命令安装 jenkins。

    docker pull jenkins/jenkins

执行以下命令运行 Jenkins。

    docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins

    或者

    mkdir -p /var/jenkins_home && chmod -R 777 /var/jenkins_home

    docker run -itd  -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins

后一种方式会在 docker 宿主机上自动创建一个叫 'jenkins_home' 的卷。容器无论停止、重启或删除，都会保留下来。

使用下面的命令查看容器启动日志。

    docker logs -f jenkins

参考：[https://github.com/jenkinsci/docker/](https://github.com/jenkinsci/docker/blob/master/README.md)
