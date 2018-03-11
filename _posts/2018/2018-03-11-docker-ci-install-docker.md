---
layout: post
title: docker 持续集成（1） - 安装 docker
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

### 安装 docker

以 centos 7 为例。官方文档：官方文档：[https://docs.docker.com/](https://docs.docker.com/install/linux/docker-ce/centos/) 。

### 第一步，卸载 docker 的旧版本。

如果没有旧版本可省略这一步。

    sudo yum remove docker \
        docker-client \
        docker-client-latest \
        docker-common \
        docker-latest \
        docker-latest-logrotate \
        docker-logrotate \
        docker-selinux \
        docker-engine-selinux \
        docker-engine

卸载后，/var/lib/docker 内的 images, containers, volumes, 以及自定义配置文件不会被删除。如果需要删除，使用以下命令。

    sudo rm -rf /var/lib/docker

### 第二步，安装 docker。

安装 docker 有三种方式：

1.设置 Docker’s repositories，推荐。

2.下载 RPM package，手动安装，手动更新，适合无法访问互联网的服务器。

3.在测试和开发环境中，使用  convenience scripts 安装。

下面介绍下第一种安装方法。

首先安装依赖包。

    sudo yum install -y yum-utils \
        device-mapper-persistent-data \
        lvm2

然后使用下面的命令设置 stable repository。

    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo

使用下面的命令安装最新的 docker CE。

    sudo yum install docker-ce

docker 安装成功后，系统中增加了一个 docker group，但是该 group 下没有用户。如果希望用非 root 用户执行 docker 命令，需要新增一个用户，加入到 docker group 中。参考：[https://docs.docker.com/](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) 。

生产系统下最好是安装指定版本的 docker，使用下面的命令列出 docker 可用的版本，然后选择安装。

    yum list docker-ce --showduplicates | sort -r

使用下面的命令安装指定版本。

    sudo yum install <FULLY-QUALIFIED-PACKAGE-NAME>

启动 docker。

    sudo systemctl start docker

使用下面的命令，验证是否安装、运行成功。这条命令会下载一个测试 image，并在容器中运行，运行成功后会在控制台打印信息并退出。

    sudo docker run hello-world

配置 docker 开机启动。

    开机启动
    sudo systemctl enable docker
    
    禁止开机启动
    sudo systemctl disable docker

