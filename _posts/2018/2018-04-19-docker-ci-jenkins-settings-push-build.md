---
layout: post
title: docker 持续集成（7） - 配置 jenkins，实现自动构建
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

之前我们在 [配置 jenkins，实现每日构建、发布、打标签](http://zhuxulu.com/2018/03/docker-ci-jenkins-settings-nightly-build-deploy-tag/) 中介绍了jenkins、gitlab的基础配置，以及如何创建一个定时任务。现在我们在上篇文章的基础上建立一个通过 `gitlab webhook` 触发的任务，内容是当开发人员提交代码到仓库master分支时，自动触发jenkins的构建任务，对代码进行自动编译。

首先在工程git仓库根目录下新建一个shell脚本，取名叫 “push_build.sh”，内容如下：

    #!/bin/bash

    cd ~/easygo-web
    git pull

    docker run --rm -v "$(pwd)":/project  -v /usr/tmp/.m2:/root/.m2  -w /project maven:3.5.3-jdk-8 mvn package

这个脚本的作用是：拉取代码，用maven进行编译打包。

然后开始配置任务，新建一个任务，选择第一项的`自由风格`，取名叫 `easygo-web-push-build`，`General` 和 `源码管理` 的配置和之前一致。

在`构建触发器`中，选择`Build when a change is pushed to GitLab`，然后复制下后面的 webhook 地址。

登陆到gitlab，进入仓库的`Settings - Integrations`，将刚才复制的地址填入到URL中，`Trigger`只勾选`Push events`。如下图：

![](http://zhuxulu.github.com/assets/post-images/gitlab-integrations-push-trigger.png)

回到jenkins任务配置界面，在`构建`中，选择之前配置的 `ssh site`，`Command`中填写：

    ~/easygo-web/push_build.sh

意思是去执行`easygo-web`目录下的`push_build.sh`这个脚本。前面的`~`代表是登陆用户的home目录，可以更换为上文中所说的工作文件夹，一定要是绝对路径，并且确保jenkins系统管理中配置的ssh账号对该目录有读写执行权限。

最后保存任务，然后我们就可以往仓库中提交代码测试是否会触发自动构建了。

任务日志可在Jenkins中查看，当然也可以配置邮件通知功能，将日志通过邮件发送给相关人员。

