---
title: camunda基础篇
date: 2022-10-20 16:38:59
tags: [BPMN,camunda]
categories: [笔记]
---
# Camunda-热门工作流引擎框架
> Lecture：波哥

![image-20220902232358815](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220902232358815.png)

# 一、工作流相关介绍

**BPM**(BusinessProcessManagement)，业务流程管理是一种管理原则，通常也可以代指BPMS(BusinessProcessManagementSuite)，是一个实现整合不同系统和数据的流程管理软件套件.

**BPMN**(BusinessProcessModelandNotation)是基于流程图的通用可视化标准。该流程图被设计用于创建业务流程操作的图形化模型。业务流程模型就是图形化对象的网状图，包括活动和用于定义这些活动执行顺序的`流程设计器`。BPMN2.0正式版本于2011年1月3日发布，常见的`工作流引擎`如：Activiti、Flowable、jBPM 都基于 BPMN 2.0 标准。

然后来看看BPM的发展历程：

![image-20220830005000114](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220830005000114.png)

# 二、Camunda介绍

官网地址：https://camunda.com/

中文站点：http://camunda-cn.shaochenfeng.com/

下载：https://downloads.camunda.cloud/release/camunda-bpm/run/7.15/

案例地址：[Congratulation! | docs.camunda.org](https://docs.camunda.org/get-started/quick-start/complete/)

前期准备工作: JAVA1.8以上的JRE或JDK

## 1.Camunda Modeler 

Camunda Modeler 是Camunda 官方提供的一个流程设计器，用于编辑流程图以及其他模型【表单】，也就是一个流程图的绘图工具。可以官方下载，也可以在提供给大家的资料中获取。获取后直接解压缩即可，注意：解压安装到非中文目录中!!!

![image-20220901105936567](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901105936567.png)



启动的效果：

![image-20220901110007447](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901110007447.png)



## 2.Camunda BPM

下载地址 https://camunda.com/download/

Camunda BPM 是Camunda官方提供的一个`业务流程管理`平台,用来管理，部署的流程定义、执行任务，策略等。下载安装一个Camunda平台，成功解压 Camunda 平台的发行版后，执行名为start.bat（对于 Windows 用户）或start.sh（对于 Unix 用户）的脚本。此脚本将启动应用程序服务器。

![image-20220901110225636](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901110225636.png)

打开您的 Web 浏览器并导航到http://localhost:8080/以访问欢迎页面，Camunda的管理平台。



![image-20220718162641800](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220718162641800.png)



登录成功的主页：

![image-20220718162726028](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220718162726028.png)







## 3.入门案例

### 3.1 创建简单流程

我们先通过 Modeler 来绘制一个简单流程

1.) 创建流程：选择 BPMN diagram (Camunda Platform)

![image-20220901110718700](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901110718700.png)

2.) 创建开始节点：并设定节点名称

![image-20220901110850675](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901110850675.png)

3.) 创建服务节点：设置处理方式

![image-20220901110926577](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901110926577.png)

![image-20220901111045576](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111045576.png)

我们切换节点的类型为 `service Task`

![image-20220901111125584](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111125584.png)

![image-20220901111141656](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111141656.png)

然后我们需要配置`刷卡付款`节点，服务类型有很多执行的方法，这次我们使用“external（外部）”任务模式。

![image-20220901111330988](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111330988.png)

具体配置内容为

![image-20220901111419588](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111419588.png)



4.) 添加结束节点

![image-20220901111521063](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111521063.png)



5.) 配置流程参数

点击画布的空白处，右侧的面板会显示当前流程本身的参数,这里我们修改id为*payment-retrieval*，id是区分流程的标识然后修改Name 为“付款流程”最后确保 `Executable`是勾选的，只有`Executable`被勾选，流程才能执行

![image-20220901111725855](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901111725855.png)



### 3.2 外部任务

在上面设计的流程图，`刷卡付款`节点的处理是外部任务，Camunda 可以使多种语言实现业务逻辑，我们以Java为例来介绍。

添加相关的依赖：

```xml
    <dependencies>
		<dependency>
			<groupId>org.camunda.bpm</groupId>
			<artifactId>camunda-external-task-client</artifactId>
			<version>7.15.0</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-simple</artifactId>
			<version>1.6.1</version>
		</dependency>
		<dependency>
			<groupId>javax.xml.bind</groupId>
			<artifactId>jaxb-api</artifactId>
			<version>2.3.1</version>
		</dependency>
	</dependencies>

```

编写处理的业务逻辑的代码

```java
import org.camunda.bpm.client.ExternalTaskClient;

import java.awt.*;
import java.net.URI;

public class Demo01 {
    public static void main(String[] args) {
        ExternalTaskClient client = ExternalTaskClient.create()
                .baseUrl("http://localhost:8080/engine-rest")
                .asyncResponseTimeout(10000) // 长轮询超时时间
                .build();
        // 订阅指定的外部任务
        client.subscribe("charge-card")
                .lockDuration(1000)
                .handler(((externalTask, externalTaskService) -> {
                    // 获取流程变量
                    String item = (String) externalTask.getVariable("item");
                    Long amount = (Long) externalTask.getVariable("amount");
                    System.out.println("item--->"+item + "  amount-->" + amount);
                    try {
                        Desktop.getDesktop().browse(new URI("https://docs.camunda.org/get-started/quick-start/complete"));
                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                    // 完成任务
                    externalTaskService.complete(externalTask);
                })).open();
    }
}

```

运行该方法即可，当流程处理时会执行相关逻辑。



### 3.3 部署流程

接下来我们就可以来部署上面定义的流程了。使用 Camunda Modeler 部署流程，点击工具栏中的部署按钮可以将当前流程部署到流程引擎，点击部署按钮，输入Deployment Name 为 “Payment” ，输入下方REST Endpoint 为http://localhost:8080/engine-rest ，然后点击右下角Deploy部署


部署操作：

![image-20220901102738775](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901102738775.png)

部署的时候报错：原因是安装路径中有中文

![image-20220901101904825](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901101904825.png)



部署成功：

![image-20220901102705446](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901102705446.png)



然后在BPM中我们可以查看部署的流程：

![image-20220901112401741](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901112401741.png)





### 3.4 启动流程

我们使用Rest API发起流程，所以需要一个接口测试工具（例如：Postman），或者也可以使用电脑自带的curl

curl执行如下命令

```curl
curl -H "Content-Type: application/json" -X POST -d '{"variables": {"amount": {"value":555,"type":"long"}, "item": {"value":"item-xyz"} } }' http://localhost:8080/engine-rest/process-definition/key/payment-retrieval/start

```

postman方式处理

在url中输入：http://localhost:8080/engine-rest/process-definition/key/payment-retrieval/start 通过`POST`方式提交，提交的方式是`JSON` 数据，具体内容为：

```json
{
	"variables": {
		"amount": {
			"value":555,
			"type":"long"
		},
		"item": {
			"value": "item-xyz"
		}
	}
}

```

![image-20220901112634925](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901112634925.png)

然后我们点击发送，操作成功可以看到如下的返回信息

![image-20220901112709320](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901112709320.png)



同时任务执行后我们在控制台可以看到相关的信息

![image-20220901112810150](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220901112810150.png)





# 三、案例扩展

上面的案例过于简单，我们添加不同的任务节点和网关来丰富下

## 1. 用户任务



### 1.1 添加节点

 我们在上面的案例中添加一个`用户任务`来处理流程。

![image-20220903233945199](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220903233945199.png)

点击刚刚创建的`批准付款`节点，然后通过扳手设置节点的类型为`用户任务`(User Task)

![image-20220903234050669](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220903234050669.png)



然后设置节点的审批人为`demo`

![image-20220903234231682](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220903234231682.png)



### 1.2 配置表单

在`用户节点`处我们绑定表单数据。然后创建表单相关的字段，并添加对应的描述信息。

![image-20220904001026418](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904001026418.png)

> amount字段：
>
> ​     ID:amount
>
> ​     Type:long
>
> ​     Label:金额
>
> Item字段：
>
> ​     ID:item
>
> ​     Type:string
>
> ​     Label:项目
>
> Approved字段：
>
> ​     ID:approved
>
> ​     Type:boolean
>
> ​     Label:是否同意



### 1.3 部署流程

流程定义好之后我们就可以部署流程了。直接在`Camunda Modeler`工具栏上的上传按钮将流程上传到流程引擎中。部署后在`Camunda Web`中查看部署的流程。

![image-20220904001636662](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904001636662.png)





### 1.4 测试流程

打开任务列表（http://localhost:8080/camunda/app/tasklist/），使用 demo / demo 登录。点击右上角的 `Start process` ，在弹出的对话框中选择“付款流程”.

![image-20220904001816475](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904001816475.png)

这时会弹出编辑流程变量的对话框，可以通过点击 Add a variable 按钮添加变量，这次我们先不添加，直接点击右下角` Start `启动流程。

![image-20220904001902564](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904001902564.png)

这时，在任务列表应该就能看到刚创建的人工任务了，如果没有可以手动刷新一下

![image-20220904001953028](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904001953028.png)



到这儿这个`用户任务`节点的人工审核就应该要处理了，我们在下一个案例中继续这个案例，我们加入排他网关来处理。





## 2.排他网关

我们将使用**排他网关**(*Exclusive Gateways*)，为流程添加分支，仅在金额足够大的时候进行人工审核.

### 2.1 添加网关节点

首先打开`Camunda Modeler `，在左侧的工具架中找到网关（菱形），将它拖动到“付款请求”和“刷卡付款”之间，将“批准付款”向下移动再添加一个网关，调整流程，最后看起来类似这样：

![image-20220904004632887](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904004632887.png)



### 2.2 配置网关

接下来，我们选择“<1000”的连线，打开属性面板，选择“Condition Type”为“Expression”，这里我们使用[JAVA统一表达式语言](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)编写条件，这里要做的是在金额小于1000时直接刷卡付款，则输入表达式 `${amount<1000}`

![image-20220904004725626](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904004725626.png)

同样的，对另一条线也进行配置，表达式为`${amount>=1000}`

![image-20220904004747234](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904004747234.png)

然后`是否批准`的排他网关节点我们也需要处理下

![image-20220904004845594](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904004845594.png)

![image-20220904004910392](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904004910392.png)



### 2.3 部署流程

部署流程和上面的操作是一样的。

![image-20220904005023089](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904005023089.png)



### 2.4 测试操作

打开任务列表（http://localhost:8080/camunda/app/tasklist/），使用 demo / demo 登录,点击右上角的 Start process ，在弹出的对话框中选择“付款流程”,上面例子中，我们直接点击 Start，但这次我们要增加几个变量来测试动态的流程。

![image-20220904005221732](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904005221732.png)

试着更改 `amount` 的值，查看对流程执行顺序的影响





## 3.决策自动化

在上面的案例中我们在审批时是通过`用户任务`结合表单来做的审批，本案例我们来看看我们通过`DMN`为流程添加一个业务规则来处理

### 3.1 添加业务规则

打开 Camunda Modeler，点击 “批准付款”，在右面的扳手菜单中，将类型改为“**Business Rule Task** ”（业务规则任务）

![image-20220904113549016](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904113549016.png)

下一步，将属性面板中的 `Implementation` 选择为`DMN`，输入 Decision Ref 为 `approve-payment`，为了将决策的结果存入流程变量，我们还需要编辑结果变量Result Variable 为`approved`，结果类型 Map Decision Result 选择为 `singleEntry `，最后结果如下：


![image-20220904113700661](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904113700661.png)



### 3.2 创建DMN表

点击 `File > New File > DMN Diagram`创建一个新的DMN,现在画布上已经存在一个决策元素了，选择它，然后在右侧属性面板中更改`Id`和`Name`，这里的Id需要和流程中的`Decision Ref`属性一致，这次我们输入Id为`approve-payment`

![image-20220904113803219](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904113803219.png)

接下来，点击决策元素左上角的表格按钮，创建新的DMN表.

![image-20220904113905505](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904113905505.png)



### 3.3 编辑DMN表

首先编辑输入参数，在本例中，为了简单，我们依据项目名进行判断，规则可以使用 `FEEL 表达式`、`JUEL`或者 `Script` 书写，详情可以阅读 https://docs.camunda.org/manual/latest/user-guide/dmn-engine/expressions-and-scripts/

双击表格中的*Input*，编辑第一个输入参数

![image-20220904114105012](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114105012.png)

下面，设置输入参数，双击*Output*编辑

![image-20220904114133016](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114133016.png)

下面，我们点击左侧的蓝色加号，添加一些规则，最后类似这样：

![image-20220904114210501](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114210501.png)



### 3.4 部署DMN表

点击底部的部署按钮，将DMN部署到流程引擎中

![image-20220904114317458](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114317458.png)



### 3.5 流程案例测试

现在打开 http://localhost:8080/camunda/app/cockpit/ ，使用demo/demo登录，可以看到决策定义增加了一个，点击进去可以看到刚才编辑的DMN.

![image-20220904114413678](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114413678.png)

点击进去可以看到对应的决策信息

![image-20220904114452196](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114452196.png)



然后我们部署流程然后启动流程

![image-20220904114623954](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114623954.png)

该流程决策输出的`approved`为true

![image-20220904114740904](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114740904.png)



更改下输入的参数

![image-20220904114822563](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114822563.png)

该决策中输出的`approved`为false

![image-20220904114913062](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904114913062.png)

好了~到此Camunda的基础入门案例我们就讲解到这里，需要获取更多Camunda相关课程视频的可以继续关注波哥的B账号，或者添加波哥微信(boge_java/boge3306)

![image-20220904115333970](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220904115333970.png)

