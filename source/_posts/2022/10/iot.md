---
title: iot边缘计算
date: 2022-09-24 19:25:51
tags: [边缘计算]
categories: [笔记]
---
## 稳定可靠的通信技术

### 有线通信技术

优点：稳定性高，可靠性强

缺点：受限于传输媒介

以太网:最通用的通信协议标准，标准以太网，快速以太网，10G以太网

RS—232：个人计算机DB9 or DB25	不平衡传输方式，单边通信	传输距离不超过20m	1V1

RS-485：可以实现联网	平衡传输，差分传输方式	几百到上千	1Vn

M-Bus：户用仪表总线，远程抄表，低成本组网

PLC：电力线通信，表表->工业网关 

### 无线通信技术

2G,3G,4G：蜂窝移动通信

Bluetooth:2.4-2.48GH波段的无线电波，速率1Mbps，10cm-10m，速率快、低功耗、安全性高，节点少不利于多点布控

Wi-Fi：连接到无线局域网，速度快

ZigBee：低功耗局域网协议	物体遮挡衰减厉害  

Z-Wave：用于住宅。30-100m	网络简单，速率较低

### lpwa低功耗广域网

SigFox：低功耗，ISM射频频段

NB-lot：窄频网络，可部署在蜂窝网络 

LoRa：开源MAC层协议

![image-20220923194944959](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923194944959.png)

![image-20220923195011645](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923195011645.png)

## OceanConnect

![image-20220923195209078](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923195209078.png)

统一接口，方便拓展

### MQTT和HTTP

消息队列遥测传输：Message Queuing Telemetry Transport

IBM开发的基于TCP/IP的即时通讯协议

采用订阅发布的模式，长连接方式

优点：协议简单，轻量级，消息可以短至两字节，对终端的硬件配置要求低

智慧家庭

CoAP：Constrained Application Protocol

专用于资源受限设备的通信，NB-lot/LoRa

从http发展而来，采用请求响应

最小4字节

### 对比

![image-20220923195746901](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923195746901.png)

物联网平台层次

![image-20220923195812117](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923195812117.png)

接入无关，可靠性，安全性，弹性伸缩，能力开放

![image-20220923200007426](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923200007426.png)

设备管理：网关管理，REST和HTTPS下发各类配置管理命令，DM Server通过Coap/HTTPS将命令传给网关或者其他直连设备，完成设备的配置管理

鉴权：创建传感器鉴权，设备接入鉴权，上报数据鉴权

规则引擎：规则绑定，满足条件，自动执行动作

Portal：SP，OSS，Operation

### 平台架构

![image-20220923201228365](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923201228365.png)

IoCM：iot联接管理模块，是平台最重要的模块，支持联接状态管理和控制命令转发

DM Server：设备管理服务器，物联网管理和升级

Rule Engine：规则引擎，用户设定规则满足业务需求

MongoDB：用户信息数据库

CIG：云网关

南向的终端设备可以通过CIG的协议适配连接平台

平台通过API Server接入北向的iot应用服务器

业务流程：

![image-20220923201805426](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20220923201805426.png)

### NB-iot

是一项基于窄带的通信技术

HSS归属用户服务器：存储用户信息的核心数据库，主要用来保存用户签约信息

EPC核心网：HSS归属用户服务器，MME信令处理部分，PGW：PDN网关，SGW服务网关

应用层：web界面