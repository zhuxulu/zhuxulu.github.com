---
layout: post
title: docker 持续集成（5） - 安装 gitlab
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

#### 安装 gitlab

执行以下命令安装 gitlab。

    docker pull gitlab/gitlab-ce

执行以下命令运行 gitlab。

    sudo docker run --detach \
    --hostname gitlab.1886.org\
    --publish 443:443 --publish 8000:80 --publish 2200:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

参考：[https://docs.gitlab.com/omnibus/docker/](https://docs.gitlab.com/omnibus/docker/)

