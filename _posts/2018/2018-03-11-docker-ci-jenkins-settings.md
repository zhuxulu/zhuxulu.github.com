---
layout: post
title: docker 持续集成（5） - 配置 jenkins
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

#### 配置 测试服务器、工程源代码、jenkins，实现持续集成。

我们将 docker 宿主机作为测试服务器。这里实现说明一下，我的环境是自己电脑的虚拟机，jenkins 没有互联网地址，因此不能使用 git 服务器的 hook 功能，其实也没有必要每次 push 时进行编译构建，那么换个思路，由 jenkins 定时构建，构建时调用宿主机的 shell 脚本，脚本里执行更新代码、编译、发布等操作，这样也是可行的。下面是具体的操作步骤。

首先，在 docker 宿主机上安装 git。

    yum install git -y

然后，在工程源码文件夹根目录下，新建一个 ci.sh 文件，内容如下：

    #!/bin/bash

    # 停止并删除 tomcat 容器。
    docker stop $(docker ps -qa --filter name='tomcat')

    # 进入工程源码文件夹，删除 target 编译目录，然后 git pull 更新代码。
    cd ~/project && rm -rf ./target && git pull

    # 编译工程
    docker run --rm --name tomcat  -v "$(pwd)":/project  -v /usr/tmp/.m2:/root/.m2  -w /project maven:3.5.3-jdk-8 mvn package

    # 将编译后的 war 包映射到 tomcat 容器中，并启动运行 tomcat 容器。
    docker run -d --rm -p 8888:8080 -v $PWD/target/easygo.war:/usr/local/tomcat/webapps/easygo.war --name tomcat  tomcat:7.0.85-jre7

其次，在当前用户的 home目录下，clone 工程源代码，如果有登陆验证，可能需要在 url 加上账号密码，或者换成 ssh 方式。

    cd ~/
    git clone https://gitee.com/zhuxulu/project.git

最后，对 jenkins 进行配置，实现定时构建。配置方法如下。

在 credentials - system - global credentials 中，新增一个凭证。填写 docker 宿主机的用户名和密码。

![](http://zhuxulu.github.com/assets/post-images/jenkins-credentials.png)

在系统管理 - 系统设置 - SSH remote hosts 中，填入 docker 宿主机的 ip 地址和端口，Credentials 选择刚才我们填写的凭证。

![](http://zhuxulu.github.com/assets/post-images/jenkins-ssh-remote-hosts.png)

新建一个任务，在“构建触发器”中，选中 Build periodically，按照规则填写，例如 H/60 * * * * 为每60分钟触发一次。

![](http://zhuxulu.github.com/assets/post-images/jenkins-task-bulid-periodically.png)

在任务的“构建”中，增加构建步骤，SSH site 选择刚才我们填写的 docker 宿主机地址，Command 中填写 “~/project/ci.sh”

![](http://zhuxulu.github.com/assets/post-images/jenkins-task-build-step.png)

保存任务，然后点击“立即构建”，构建成功后，可以打开浏览器访问 tomcat 了。


