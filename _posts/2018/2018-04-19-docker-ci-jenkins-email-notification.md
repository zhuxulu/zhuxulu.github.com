---
layout: post
title: docker 持续集成（8） - 配置 jenkins 邮件通知
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
- email
---

之前我们已经介绍了 [配置 jenkins，实现每日构建、发布、打标签](http://zhuxulu.com/2018/03/docker-ci-jenkins-settings-nightly-build-deploy-tag/) 和[配置 jenkins，实现自动构建](http://zhuxulu.com/2018/04/docker-ci-jenkins-settings-push-build/)。虽然实现了自动化，但是任务是否执行成功，执行结果怎么样，具体报告的内容，都需要我们登陆到jenkins去查看，如果某个开发人员提交的代码导致工程编译不通过，那么我们就不能及时发现。下面我们通过配置邮件通知的方式实现任务执行完立即发送邮件通知相关人员，这样整个过程看起来就非常完整了。

首先我们要安装`Email Extension Plugin`插件，之前已经安装过了，那么转到`系统管理 - 系统设置 - Extended E-mail Notification` 中，填写相关的邮件服务器信息：

![](http://zhuxulu.github.com/assets/post-images/jenkins-email-settings.png)


然后我们需要修改之前创建的两个任务，以`easygo-web-nightly-build-deploy-tag`为例：

在`构建后操作`中，点击`增加构建后操作步骤`，选择`Editable Email Notification`。

- `Project From` 填写发件箱邮件地址。
- `Project Recipient List` 填写邮件接收人地址，英文逗号隔开，可以填写项目管理者的邮箱。

![](http://zhuxulu.github.com/assets/post-images/jenkins-email-recipient.png)

- `Content Type` 选择 `HTML(text\html)`
- `Default Subject` 邮件标题，填写：构建通知:${BUILD_STATUS} - ${PROJECT_NAME} - Build # ${BUILD_NUMBER} !
- `Default Content` 邮件内容。
- `Attach Build Log` 选择 `Attach Build Log`，将日志当作附件附加到邮件中。

邮件内容模板：

    ```
    <hr/>
    构建状态：$BUILD_STATUS<br/>
    项目名称：$PROJECT_NAME<br/>
    构建编号：第$BUILD_NUMBER次每日构建<br/>
    构建时间：$BUILD_TIMESTAMP<br/>
    触发原因：${CAUSE}<br/>
    GIT 仓库：${GIT_URL}<br/>
    GIT 分支：${GIT_BRANCH}<br/>
    构建日志：<a href=”${BUILD_URL}console”>${BUILD_URL}console</a><br/>
    工作目录：<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a><br/>
    <hr/>
    ${JELLY_SCRIPT,template="html"}<br/>
    ${JELLY_SCRIPT,template="html"}<br/>
    ```

保存任务后我们可以手动触发一下任务，看看是否能够收到邮件，构建成功的邮件通知如下：

![](http://zhuxulu.github.com/assets/post-images/jenkins-email-success.png)

上文中出现了一些以`$`开头的变量，这些变量是Jenkins的内置变量，可以从这里查看：[jenkins地址](http://jenkins地址/env-vars.html/)或者[wiki.jenkins.io](https://wiki.jenkins.io/display/JENKINS/Building+a+software+project#Buildingasoftwareproject-below)