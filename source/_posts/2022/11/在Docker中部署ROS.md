---
title: 在Docker中部署ROS
date: 2022-11-09 22:01:43
tags: [Docker, ros]
categories: [笔记]
---
# 在Docker中部署ROS

——提高开发环境鲁棒性

## 安装之前

### Docker容器会占用多大存储空间

Docker通过images创造container，在container中部署和开发应用。

ROS官方已经提供了支持Docker的ros images，镜像已经配置好了ros的开发环境。通过命令docker pull osrf/ros：melodic-desktop-full拉取镜像，如果是包含ros-core的镜像，大概是400M，如果包含rviz，gazebo等全套的destktop-full版本，大概是5G左右，占据的空间还是很可观的。

查看容器与镜像大小

>  docker system df -v

![image-20221109153440000](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221109153440000.png)

### 查看docker里面可视化界面

宿主机开启xhost, 使能宿主机接收其他客户端的显示需求
 `xhost +`

docker创建容器时参数设置xserver挂载地址即可
 `-e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix`

### docker里面的程序如何与主机的程序通讯

采用socket通信

docker创建容器时，默认分配了bridge网络，这样所有的容器都在同一个网段，是能够相互通信的。

如果想某些容器之间不能通信，通过自定义网络就能实现。

假如宿主机是linux系统，有一个监听8000端口的其他进程

在宿主机上执行ifconfig命令，可以发现有一个docker0网卡，注意观察其网段和docker network inspect bridge的网段是一致的，宿主机在此网段下也分配了一个ip，一般是网段中的第一个ip，假如是192.168.10.1。

1）假如用默认的bridge模式启动容器，即在启动时不指定network参数或者指定了--network bridge，

在容器中执行ifconfig命令，可以发现容器的ip也在上面的网段中，假如是192.168.10.2。

在容器中，ping 192.168.10.1，可以ping通。在宿主机中ping 192.168.10.2，可以ping通。

在容器中，可以用192.168.10.1:port的方式访问宿主机的服务。

网络中的其它主机能否ping通这些容器？
 roscore在容器中启动后，宿主机能否启动？可以，这相当于是两台不同的机器，如果共享一个roscore，需要设置两个的bashrc，ros_master_url参数。

2）假如容器用host网络模式启动，即在启动时添加了--netwok host参数，

那么容器会和宿主机共享网络，直接telnet 127.0.0.1 8000可以telnet通信。

## docker可视化

docker中开启GUI原理：
 xhost 是用来控制X server访问权限的。通常当你从hostA登陆到hostB上运行hostB上的应用程序时，做为应用程序来说，hostA是client,但是作为图形来说，**是在hostA上显示的，需要使用hostA的Xserver**,所以hostA是server。因此在登陆到hostB前，需要在hostA上运行xhost +来使其它用户能够访问hostA的Xserver.

**总结：docker中显示client通过映射将对Xserver的请求透传到主机端DISPALY，DISPLAY对应主机端的显示接口，主机端也使能接收，完成显示。**

xhost + **是使所有用户都能访问Xserver.**
 xhost + ip使ip上的用户能够访问Xserver.
 xhost + nis:user@domain使domain上的nis用户user能够访问
 xhost + inet:user@domain使domain上的inet用户能够访问。

### 运行xserver

终端输入
`xhost +`

- 若成功启动，则跳到第4.2继续执行。
- 若显示`xhost: unable to open display`，则说明没有安装vncserver。

### 安装vncserver

进入root，确认下vncserver确实没装（我也不明白为什么要进入root，但是照做就行）
 `su root`
 `vncsever`
 若提示`bash: vncserver: command not found`则需要安装vnc。

退出root（或者重新开一个终端），安装vnc
 `sudo apt-get install tigervnc-standalone-server tigervnc-viewer`
 上面命令是我测试没问题的，网上还有其它安装的包

**如果上面的命令能安装好vnc，就不用下面这些了
 `sudo apt-get install x11-xserver-utils`
 上面这个包系统已经默认装好了，但是仍然不能执行`xhost +`

`sudo apt install xfce4 xfce4-goodies`
 `sudo apt install tightvncserver`
 上面这个包没有测试过，感觉似乎可以。

`sudo apt-get install tigervnc-server`
 上面这个包没有测试过，感觉似乎也可以。

如果vncserver报错，查看日志文件，并逐个解决

安装完后就是配置VNC，感觉这个设置挺复杂的，采用了网上最简单的操作方式

 安装好vnc后，进入root启动vnc
 `su root`
 `vncserver`
 第一次运行会让你输入很多密码，输入就行，一路yes～

启动容器，测试gazebo和rviz，都可以正常打开

## 多ros容器通信

### 第一个终端——roscore：

①docker命令新建一个容器（带GUI环境变量）

```bash
docker run -it \
    --env="DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    osrf/ros:kinetic-desktop-full \
    /bin/bash
```

命令解释
 --env="DISPLAY" ：开启显示GUI界面
 --env="QT_X11_NO_MITSHM=1" ：采用X11的端口1进行显示
 --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" ：映射显示服务节点目录
 osrf/ros:kinetic-desktop-full：容器从镜像osrf/ros:kinetic-desktop-full创建
 /bin/bash：运行命令bash

②创建完容器后，命令行自动进入了docker，接着运行roscore

`export ROS_MASTER_URI=http://172.17.0.1:11311`

`export ROS_IP=172.17.0.1`

`source /opt/ros/kinetic/setup.bash`
`roscore`

### 第二个终端——turtlesim_node

①先查看之前新建的容器名称
 `docker ps --all`
 ②进入容器中
 `docker exec -it <container_name> /bin/bash`
 ③运行turtlesim_node

`export ROS_MASTER_URI=http://172.17.0.1:11311`

`export ROS_IP=172.17.0.2`

 `source /opt/ros/kinetic/setup.bash`
 `rosrun turtlesim turtlesim_node`

![img](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/20587097-f9e2481d64ba9892.png)

### 第三个终端——键盘控制：

 ①进入同一个容器中
 `docker exec -it <container_name> /bin/bash`
 `source /opt/ros/kinetic/setup.bash`
 ②运行遥控键盘

`export ROS_MASTER_URI=http://172.17.0.1:11311`

`export ROS_IP=172.17.0.3`

`rosrun turtlesim turtle_teleop_key`
 此时即可控制小海龟运动

![img](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/20587097-97dad70fedf029a1.png)



