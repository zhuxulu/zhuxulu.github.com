---
layout: post
title: docker 持续集成（6） - 配置 jenkins，实现每日构建、发布、打标签
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


我们将 docker 宿主机作为测试服务器。通过 jenkins、gitlab 实现工程的每日定时构建、发布、给git仓库打标签的功能。

前期准备工作，首先在 docker 宿主机上安装 git。

    yum install git -y

然后，在工程git仓库根目录下新建一个shell脚本，取名叫 “build_deploy.sh”，内容如下：

    #!/bin/bash

    docker stop $(docker ps -qa --filter name='tomcat')

    sleep 3

    cd ~/easygo-web && git pull

    # 编译工程
    docker run --rm -v "$(pwd)":/project  -v /usr/tmp/.m2:/root/.m2  -w /project maven:3.5.3-jdk-8 mvn package

    # 将编译后的 war 包映射到 tomcat 容器中，并启动运行 tomcat 容器。
    docker run -d --rm -p 8089:8080 -v $PWD/target/easygo-web.war:/usr/local/tomcat/webapps/easygo.war --name tomcat  tomcat:7.0.85-jre7

这个脚本的作用是：删除运行中的tomcat容器，git pull更新工程代码，maven编译代码，将编译后的war包映射到tomcat容器的目录，启动tomcat。

然后我们登陆到测试服务器，生成ssh key，通过ssh方式免密码下载代码。

    ssh-keygen -t rsa -C "xxxx@gmail.com" -b 4096

根据提示一路回车，最后在当前登陆用户home目录下生成一个 `.ssh` 文件夹，拷贝该文件夹下公钥 `id_rsa.pub` 文件里的内容。

然后登陆到gitlab，找到当前工程，进入工程下的 `Settings - 
Repository`, 展开 `Deploy Keys`，将拷贝的公钥填入到 `key` 里，`title` 填写该 key 的名称。`Deploy Keys` 只允许下载代码，不允许其他操作，这里我们将测试服务器的公钥作为 `Deploy Keys`，能够保证测试服务器只具有下载代码的权限，其他人登陆到测试服务器，不会对仓库进行更新代码等危险操作。

回到测试服务器，我们需要在服务器中建立一个工作文件夹，然后在该文件夹下 git clone 仓库地址。

    git clone https://gitee.com/zhuxulu/project.git

然后登陆到jenkins，对 jenkins 进行配置，实现定时构建、发布、给gitlab仓库打标签。实际上上文中的shell脚本的作用是构建、发布，jenkins只需要配置定时调用该脚本、给gitlab仓库打标签即可。配置方法如下。

在 `credentials - system - global credentials` 中，新增一个凭证，kind 选择 `Username with Password`, 填写 docker 宿主机的用户名和密码。

![](http://zhuxulu.github.com/assets/post-images/jenkins-credentials.png)

在`系统管理 - 系统设置 - SSH remote hosts` 中，填入 docker 宿主机的 ip 地址和端口，`Credentials` 选择刚才我们填写的凭证。

![](http://zhuxulu.github.com/assets/post-images/jenkins-ssh-remote-hosts.png)

以上的作用是使 jenkins 能够登陆到测试服务器，并在测试服务器上执行命令。

然后在 `credentials - system - global credentials` 中，再新增一个凭证，`kind` 选择 `GitLab API token`，然后我们用对仓库有管理权限的账号登陆到gitlab，在 `User Settings - Access Tokens` 中生成 API token，注意 token 生成后只显示一次，一定要及时复制，之后不会再显示了。将生成的 token 填入到 jenkins 的凭证中，然后保存。

之前安装jenkins时我们已经安装了gitlab插件，现在转到Jenkins的`系统管理 - 系统设置 - Gitlab` 中，填写 gitlab 的信息，`Connection name` 可以填 “gitlab”，`Gitlab host URL`填写你的gitlab地址，`Credentials` 选择刚才建立的 `GitLab API token` 凭证。

![](http://zhuxulu.github.com/assets/post-images/gitlab-connection.png)

以上的作用是使 jenkins 能够连接访问 gitlab，能够给 gitlab 的仓库进行相关操作，例如后续我们需要的打标签操作。

最后开始配置任务，新建一个任务，选择第一项的`自由风格`，取名叫 `easygo-web-nightly-build-deploy-tag`，`General` 中的 `GitLab connection` 选择之前系统设置中配置的`gitlab`。

![](http://zhuxulu.github.com/assets/post-images/jenkins-general.png)

在`源码管理`中，选择`git`，在`Repository URL`中填写仓库地址，因为仓库是私有的，因此需要在下方选择凭证。

![](http://zhuxulu.github.com/assets/post-images/jenkins-sourcemanage.png)

我们需要点击图中的 `add` 按钮，将对仓库有管理权限的账号密码填入。

![](http://zhuxulu.github.com/assets/post-images/jenkins-sourcemanage-credentials.png)

    这里的凭证我们也可以更换为 ssh key，需要在jenkins所在服务器生成，但是由于我们的Jenkins是运行在docker容器中，并且没有将docker宿主机中的`.ssh`目录映射到容器中，所以我们采用用户名密码的凭证方式，缺点是gitlab不是https方式访问，不安全。

在`构建触发器`中，我们按照定时触发的方式来执行任务，选择`Build periodically`，在`日程表`中填写:

    TZ=Asia/Chongqing
    @midnight

这段代码的意思是，午夜执行，时区设置为亚洲重庆。如下图：

![](http://zhuxulu.github.com/assets/post-images/jenkins-triger.png)


在`构建`中，选择之前配置的 `ssh site`，`Command`中填写：

    ~/easygo-web/build_deploy.sh
    
意思是去执行`easygo-web`目录下的`build_deploy.sh`这个脚本。前面的`~`代表是登陆用户的home目录，可以更换为上文中所说的工作文件夹，一定要是绝对路径，并且确保jenkins系统管理中配置的ssh账号对该目录有读写执行权限。

    注意：
    当真正去执行Jenkins的这个任务时，可能会出现`build_deploy.sh`文件没有执行权限的错误，需要在git提交时对文件权限做相应的修改，使用如下代码：
    git update-index --chmod=+x build_deploy.sh

然后我们在`构建后操作`中，点击`增加构建后操作步骤`，选择`Git Publisher`，勾选第一项`Push Only If Build Succeeds`，只有在构建成功后才操作。然后在`Tags`中的`Tag to Push`，填写tag的名称，一般情况下只能用Jenkins的内置变量，例如构建编号`$BUILD_NUMBER`，假如我们还想用日期作为tag的名称，就需要用到插件。

之前我们在安装jenkins时安装了一个`Build Timestamp Plugin`插件，这个插件的作用就是生成日期变量。在Jenkins的`系统管理 - 系统设置 - Gitlab` 中找到`Build Timestamp`，配置新的日期参数，如下图：

![](http://zhuxulu.github.com/assets/post-images/jenkins-build-timestamp.png)

然后回到刚才的任务配置界面，在`Tag to Push`中填写tag名称时，就可以使用这个变量，例如：

    nightly-$BUILD_NUMBER-$BUILD_DATE

然后勾选上`Create new tag`，`Target remote name`填写 “origin”。

最后保存任务，这个任务将会在每天的北京时间凌晨12点多运行，运行的步骤和结果是：关闭正在运行的tomcat、拉取代码、编译代码、发布到tomcat、运行tomcat。构建成功后，可以打开浏览器访问 tomcat 了。

也可以手动触发任务执行，测试下任务配置是否有问题。

![](http://zhuxulu.github.com/assets/post-images/jenkins-build-manual.png)

任务日志可在Jenkins中查看，当然也可以配置邮件通知功能，将日志通过邮件发送给相关人员。

