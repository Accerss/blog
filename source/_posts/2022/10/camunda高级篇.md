---
title: camunda高级篇
date: 2022-10-20 16:38:55
tags: [BPMN,camunda]
categories: [笔记]
---
# Camunda-热门工作流引擎框架
> Lecture：波哥

![image-20220902232358815](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220902232358815.png)

# 一、IDEA引入流程设计器

在工作流引擎中流程设计器是一个非常重要的组件，而`InterlliJ IDEA`是Java程序员用到的最多的编程工具了。前面在基础篇的介绍中我们都在通过Camunda提供的流程设计器绘制好流程图，然后需要单独的拷贝到项目中，要是调整修改不是很方便，这时我们可以在IDEA中和流程设计器绑定起来。这样会更加的灵活。

## 1.下载Camunda Model

第一步肯定是需要下载`Camunda Model` 这个流程设计器，我们前面有介绍。就是之前解压好的目录了。

![image-20220907002253021](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907002253021.png)



## 2.IDEA中配置

我们先进入`settings`中然后找到`tools`,继续找到`External Tool`.

![image-20220907002432482](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907002432482.png)

![image-20220907002649744](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907002649744.png)

最终效果

![image-20220907002735603](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907002735603.png)



## 3.编辑bpmn文件

找到您想打开的bpmn文件, 点击右键, 找到External Tools 运行camunda modler即可进行文件编写.

![image-20220907002851738](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907002851738.png)

搞定~



# 二、SpringBoot整合Camunda

## 1.官方案例说明

接下来我们看看怎么在我们的实际项目中来使用Camunda了。方式有多种，首先我们可以参考官网提供的整合案例。

![image-20220907003810268](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907003810268.png)

但是这里有个比较头疼的问题就是Camunda和SpringBoot版本的兼容性问题，虽然官方也给出了兼容版本的对照表。

![image-20220907003929928](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907003929928.png)



但是如果不小心还是会出现各种问题，比如：

![image-20220907004050862](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907004050862.png)

上面就是典型的版本不兼容的问题了。



## 2.官方Demo

为了能让我们的案例快速搞定，我们可以通过Camunda官方提供的网站来创建我们的案例程序。地址：https://start.camunda.com/

![image-20220907004531764](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907004531764.png)

生成代码后，解压后我们通过idea打开项目，项目结构

![image-20220907010229047](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907010229047.png)

相关的pom.xml中的依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.boge.workflow</groupId>
  <artifactId>camunda-project-demo</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.4.3</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>

      <dependency>
        <groupId>org.camunda.bpm</groupId>
        <artifactId>camunda-bom</artifactId>
        <version>7.15.0</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>org.camunda.bpm.springboot</groupId>
      <artifactId>camunda-bpm-spring-boot-starter-rest</artifactId>
    </dependency>

    <dependency>
      <groupId>org.camunda.bpm.springboot</groupId>
      <artifactId>camunda-bpm-spring-boot-starter-webapp</artifactId>
    </dependency>

    <dependency>
      <groupId>org.camunda.bpm</groupId>
      <artifactId>camunda-engine-plugin-spin</artifactId>
    </dependency>

    <dependency>
      <groupId>org.camunda.spin</groupId>
      <artifactId>camunda-spin-dataformat-all</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>2.4.3</version>
      </plugin>
    </plugins>
  </build>

</project>
```

属性文件的配置信息

```yaml
spring.datasource.url: jdbc:h2:file:./camunda-h2-database

camunda.bpm.admin-user:
  id: demo
  password: demo
```

然后通过启动类启动程序

![image-20220907010434493](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907010434493.png)

访问服务：http://localhost:8080/

![image-20220907010523901](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907010523901.png)





## 3.MySQL数据库

上面的例子我们数据存储在了H2这个内存型数据库，我们可以切换到`MySQL`数据库。首先我们需要导入相关的SQL脚本。位置就在我们之前下载的`Camunda Web`服务中。

![image-20220907010729453](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907010729453.png)

执行创建所有必需的表和默认索引的SQL DDL脚本。上面两个脚本都要执行。

![image-20220907011030798](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907011030798.png)

生成的相关表结构比较多，因为本身就是基于Activiti演变而来，所以有Activiti基础的小伙伴会非常轻松了。简单介绍下相关表结构的作用。

* **ACT_RE** ：'RE'表示 repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
* **ACT_RU**：'RU'表示 runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Flowable只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。
* **ACT_HI**：'HI'表示 history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
* **ACT_GE**： GE 表示 general。 通用数据， 用于不同场景下 
* **ACT_ID:**   ’ID’表示identity(组织机构)。这些表包含标识的信息，如用户，用户组，等等。

具体的表结构的含义:

| **表分类**   | **表名**              | **解释**                                           |
| ------------ | --------------------- | -------------------------------------------------- |
| 一般数据     |                       |                                                    |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                           |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                       |
| 流程历史记录 |                       |                                                    |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                     |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                     |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                   |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                         |
|              | [ACT_HI_IDENTITYLINK] | 历史的流程运行过程中用户关系                       |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                     |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                     |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                         |
| 流程定义表   |                       |                                                    |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                       |
|              | [ACT_RE_MODEL]        | 模型信息                                           |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义                                   |
| 运行实例表   |                       |                                                    |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                         |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                 |
|              | [ACT_RU_IDENTITYLINK] | 运行时用户关系信息，存储任务节点与参与者的相关信息 |
|              | [ACT_RU_JOB]          | 运行时作业                                         |
|              | [ACT_RU_TASK]         | 运行时任务                                         |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                       |
| 用户用户组表 |                       |                                                    |
|              | [ACT_ID_BYTEARRAY]    | 二进制数据表                                       |
|              | [ACT_ID_GROUP]        | 用户组信息表                                       |
|              | [ACT_ID_INFO]         | 用户信息详情表                                     |
|              | [ACT_ID_MEMBERSHIP]   | 人与组关系表                                       |
|              | [ACT_ID_PRIV]         | 权限表                                             |
|              | [ACT_ID_PRIV_MAPPING] | 用户或组权限关系表                                 |
|              | [ACT_ID_PROPERTY]     | 属性表                                             |
|              | [ACT_ID_TOKEN]        | 记录用户的token信息                                |
|              | [ACT_ID_USER]         | 用户表                                             |

 

然后我们在SpringBoot项目中导入`MySql`的依赖，然后修改对应的配置信息

```xml
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

修改`application.yaml`。添加数据源的相关信息。

```yaml
# spring.datasource.url: jdbc:h2:file:./camunda-h2-database

camunda.bpm.admin-user:
  id: demo
  password: demo
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/camunda1?serverTimezone=Asia/Shanghai
    username: root
    password: 123456
camunda:
  bpm:
    database:
      type: mysql
      schema-update: true
    auto-deployment-enabled: false # 自动部署 resources 下的 bpmn文件
```

然后启动项目，发现数据库中有了相关记录，说明操作成功

![image-20220907011831014](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220907011831014.png)





# 三、Camunda专题讲解

用了整合的基础我们就可以来完成一个流程审批的案例了

## 1.部署流程

```java
@RestController
@RequestMapping("/flow")
public class FlowController {

    @Autowired
    private RepositoryService repositoryService;

    @GetMapping("/deploy")
    public String deplopy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("部署的第一个流程") // 定义部署文件的名称
                .addClasspathResource("process.bpmn") // 绑定需要部署的流程文件
                .deploy();// 部署流程
        return deploy.getId() + ":" + deploy.getName();
    }
}
```

启动后访问接口即可

## 2.启动流程

启动流程我们通过单元测试来操作

```java
package com.boge.workflow;


import org.camunda.bpm.engine.RepositoryService;
import org.camunda.bpm.engine.RuntimeService;
import org.camunda.bpm.engine.TaskService;
import org.camunda.bpm.engine.runtime.ProcessInstance;
import org.camunda.bpm.engine.task.Task;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest(classes = Application.class)
public class ApplicationTest {

    @Autowired
    private RepositoryService repositoryService;

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    /**
     * 启动流程的案例
     */
    @Test
    public void startFlow(){
        // 部署流程
        ProcessInstance processInstance = runtimeService
                .startProcessInstanceById("1a880f27-2e57-11ed-80d9-c03c59ad2248");
        // 部署的流程实例的相关信息
        System.out.println("processInstance.getId() = " + processInstance.getId());
        System.out.println("processInstance.getProcessDefinitionId() = " + processInstance.getProcessDefinitionId());
    }


}

```



## 3.查询待办

查询待办也就是查看当前需要审批的任务，通过TaskService来处理

```java
    /**
     * 查询任务
     *    待办
     *
     *  流程定义ID:processDefinition : 我们部署流程的时候会，每一个流程都会产生一个流程定义ID
     *  流程实例ID:processInstance ：我们启动流程实例的时候，会产生一个流程实例ID
     */
    @Test
    public void queryTask(){
        List<Task> list = taskService.createTaskQuery()
                //.processInstanceId("eff78817-2e58-11ed-aa3f-c03c59ad2248")
                .taskAssignee("demo1")
                .list();
        if(list != null && list.size() > 0){
            for (Task task : list) {
                System.out.println("task.getId() = " + task.getId());
                System.out.println("task.getAssignee() = " + task.getAssignee());
            }
        }
    }
```



## 4.完成任务

```java
   /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        // 根据用户找到关联的Task
        Task task = taskService.createTaskQuery()
                //.processInstanceId("eff78817-2e58-11ed-aa3f-c03c59ad2248")
                .taskAssignee("demo")
                .singleResult();
        if(task != null ){
            taskService.complete(task.getId());
            System.out.println("任务审批完成...");
        }
    }
```











好了~到此Camunda的基础入门案例我们就讲解到这里，需要获取更多Camunda相关课程视频的可以继续关注波哥的B账号，或者添加波哥微信(boge_java/boge3306)

![image-20220904115333970](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904115333970.png)

