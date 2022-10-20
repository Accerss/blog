---
title: camunda进阶篇
date: 2022-10-20 16:38:45
tags: [BPMN,camunda]
categories: [笔记]
---
# Camunda-热门工作流引擎框架
> Lecture：波哥

![image-20220902232358815](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220902232358815.png)

# 一、ProcessEngine

&emsp;&emsp;ProcessEngine是Camunda流程引擎的核心。我们在流程中的很多具体的处理比如`流程部署`、`流程部署`、`流程审批`等操作都是通过`XXXService`来处理的。而相关的`XXXService`都是通过`ProcessEngine`来管理的。所以对于`ProcessEngine`的创建方式还是很有必要掌握的。



## 1. 通过xml配置方式

&emsp;&emsp;配置你的流程引擎的最简单的方法是通过一个叫做`camunda.cfg.xml`的XML文件。使用这个文件，你可以简单这样做:

```java
    @Test
    public void processEngine3(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        System.out.println("processEngine = " + processEngine);
    }
```

&emsp;&emsp;我们定义如下的`camunda.cfg.xml`文件。注意`camunda.cfg.xml`必须包含一个id为`processEngineConfiguration`的bean

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="processEngineConfiguration" class="org.camunda.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration">

        <property name="jdbcUrl" value="jdbc:h2:mem:camunda;DB_CLOSE_DELAY=1000" />
        <property name="jdbcDriver" value="org.h2.Driver" />
        <property name="jdbcUsername" value="sa" />
        <property name="jdbcPassword" value="" />

        <property name="databaseSchemaUpdate" value="true" />

        <property name="jobExecutorActivate" value="false" />

        <property name="mailServerHost" value="mail.my-corp.com" />
        <property name="mailServerPort" value="5025" />
    </bean>

</beans>
```

&emsp;&emsp;如果没有找到`camunda.cfg.xml`资源，默认引擎将搜索`activiti.cfg.xml`文件作为备用。如果两者都缺失，引擎就会停止运行，并打印出关于缺失配置资源的错误信息。

&emsp;&emsp;请注意，配置XML实际上是一个Spring配置。这并不意味着Camunda引擎只能在Spring环境中使用。我们只是在内部利用Spring的解析和依赖注入功能来建立引擎。

&emsp;&emsp;ProcessEngineConfiguration对象也可以使用配置文件以编程方式创建。也可以使用不同的bean id。

```java
ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault();
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource);
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);
```





## 2. JavaAPI方式

&emsp;&emsp;我们也可以通过创建正确的ProcessEngineConfiguration对象或使用一些预定义的对象，以编程方式配置流程引擎。

```java
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```

- `org.camunda.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration`
  流程引擎是以独立的方式使用的。引擎本身将负责处理事务。默认情况下，只有在引擎启动时才会检查数据库（如果没有数据库模式或模式版本不正确，会抛出一个异常）。
- `org.camunda.bpm.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration`
  这是一个用于单元测试的工具类。引擎本身将负责处理事务。默认使用H2内存数据库。该数据库将在引擎启动和关闭时被创建和删除。当使用这个时，可能不需要额外的配置（除了，当使用Job执行器（job executor）或邮件功能时）。
- `org.camunda.bpm.engine.spring.SpringProcessEngineConfiguration`
  当流程引擎被用于Spring环境时使用。



```java
    @Test
    public void processEngine1(){
        ProcessEngineConfigurationImpl cfg = new StandaloneProcessEngineConfiguration()
                .setJdbcUrl("jdbc:mysql://localhost:3306/camunda1?serverTimezone=UTC")
                .setJdbcUsername("root")
                .setJdbcPassword("123456")
                .setJdbcDriver("com.mysql.cj.jdbc.Driver")
                .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
                .setHistory("full");
        ProcessEngine processEngine = cfg.buildProcessEngine();
        System.out.println("processEngine = " + processEngine);
    }

   @Test
    public void processEngine2(){
        ProcessEngineConfiguration cfg = ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
        cfg.setJdbcUrl("jdbc:mysql://localhost:3306/camunda2?serverTimezone=UTC")
                .setJdbcUsername("root")
                .setJdbcPassword("123456")
                .setJdbcDriver("com.mysql.cj.jdbc.Driver")
                .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
                .setHistory("full");

        ProcessEngine processEngine = cfg.buildProcessEngine();
        System.out.println("processEngine = " + processEngine);
    }
```



## 3.SpringBoot项目

&emsp;&emsp;在SpringBoot项目会根据我们导入的依赖完成自动装配，从而完成ProcessEngine的自动注入。我们可以来分析下源码。

![image-20220911234908597](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220911234908597.png)

&emsp;&emsp;我们需要注意对于Camunda的相关配置。我们可以在application.yml 中配置。原因是 CamundaBpmProperties的处理。然后就是 @Import(CamundaBpmConfiguration.class) 。 在CamundaBpmConfiguration会完成相关的 ProcessEngineConfiguration 的相关注入。

![image-20220911234945653](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220911234945653.png)

![image-20220911235223373](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220911235223373.png)



这块感兴趣的小伙伴可以仔细阅读下。



## 4.Service API

&emsp;&emsp;Java API是与引擎互动的最常见方式。中心起点是`ProcessEngine`，它可以通过几种方式创建，如配置部分所述。从ProcessEngine中，你可以获得包含工作流/BPM方法的各种服务。`ProcessEngine`和`服务对象`是`线程安全`的。所以你可以为整个服务器保留对其中一个对象的引用.

![img](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/api.services.png)





```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

RepositoryService repositoryService = processEngine.getRepositoryService();
RuntimeService runtimeService = processEngine.getRuntimeService();
TaskService taskService = processEngine.getTaskService();
IdentityService identityService = processEngine.getIdentityService();
FormService formService = processEngine.getFormService();
HistoryService historyService = processEngine.getHistoryService();
ManagementService managementService = processEngine.getManagementService();
FilterService filterService = processEngine.getFilterService();
ExternalTaskService externalTaskService = processEngine.getExternalTaskService();
CaseService caseService = processEngine.getCaseService();
DecisionService decisionService = processEngine.getDecisionService();
```

**注意**：所有的服务都是`无状态的`。这意味着你可以很容易地在一个集群的多个节点上运行Camunda平台，每个节点都去同一个数据库，而不必担心哪个机器实际执行了以前的调用。对任何服务的任何调用都是无状态的，无论它在哪里执行。

每个服务的简单介绍

| 服务名称                             | 介绍                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| 仓库服务**RepositoryService**        | 提供了管理和操纵部署和流程定义的操作                         |
| 运行时服务-**RuntimeService**        | 首先可以通过一个流程定义启动多个流程实例。也能用于处理检索和存储流程变量的服务 |
| 任务服务-**TaskService**             | 围绕用户审批操作的一切都会被归纳到TaskService。比如：查询分配给用户或组的任务，创建新的独立任务。这些是与流程实例无关的任务，操纵一个任务被分配给哪个用户，或者哪个用户以某种方式参与到任务中，声称并完成一项任务。声称意味着有人决定成为该任务的受让人，意味着这个用户将完成该任务。完成意味着 “完成任务的工作”等 |
| 身份服务-**IdentityService**         | 是非常简单的。它允许对组和用户进行管理（创建、更新、删除、查询…）。重要的是要理解，核心引擎实际上在运行时并不对用户进行任何检查 |
| 表单服务-**FormService**             | 一个可选的服务。提供了表单功能                               |
| 历史服务-**HistoryService**          | 暴露了引擎收集的所有历史数据。当执行流程时，引擎可以保留很多数据（这是可配置的），如流程实例的开始时间、谁做了哪些任务、完成任务花了多长时间、每个流程实例遵循的路径等。该服务主要暴露了访问这些数据的查询功能。 |
| 管理服务-**ManagementService**       | 它允许检索关于数据库表和表元数据的信息。此外，它暴露了查询功能和Job的管理操作。Job在引擎中被用于各种事情，如定时器、异步延续、延迟暂停/激活等。 |
| 过滤器服务-**FilterService**         | 允许创建和管理过滤器。过滤器是像任务查询一样的存储查询。例如，过滤器被任务列表用来过滤用户任务 |
| 外部任务服务-**ExternalTaskService** | 提供对外部任务实例的访问。外部任务代表在外部处理的工作项目，独立于流程引擎。 |
| 案例服务-**CaseService**             | 与运行时服务（**RuntimeService**）类似，但用于案例实例。它处理启动案例定义的新案例实例并管理案例执行的生命周期。该服务也被用来检索和更新案例实例的流程变量。 |
| 决策服务-**DecisionService**         | 允许评估部署在引擎中的决策。它是评估独立于流程定义的业务规则任务中的决策的一种选择。 |





# 二、任务分配

## 1.固定分配

&emsp;&emsp;在指派`用户任务`的审批人时。我们是直接指派的固定账号。但是为了保证流程设计审批的灵活性。我们需要各种不同的分配方式，所以这节我们就详细的来介绍先在Camunda中我们可以使用的相关的分配方式



&emsp;&emsp;固定分配就是我们前面介绍的，在绘制流程图或者直接在流程文件中通过Assignee来指定的方式.

![image-20220912003431387](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912003431387.png)



&emsp;&emsp;这种方式就非常简单。不再过多赘述。



## 2.值表达式

&emsp;&emsp;**值表达式 Value expression**: 解析为一个值。默认情况下，所有流程变量都可以使用。（若使用Spring）所有的Spring bean也可以用在表达式里。例如:

```java
${myVar}
${myBean.myProperty}
```

![image-20220912003628341](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912003628341.png)



&emsp;&emsp;然后可以演示下这个案例，先部署该流程。

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程")
                .addClasspathResource("flow/1-01-任务分配.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```

&emsp;&emsp;然后在启动流程实例。启动流程实例后会进入到`人事审批`这个节点，有`值表达式`的存在，我们需要在启动的过程中就给其赋值。

![image-20220912004447956](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912004447956.png)

绑定流程变量的操作

```java
    /**
     * 启动流程实例
     */
    @Test
    public void startFlow(){
        String processInstanceId = "Process_0uiy3j1:1:712d273a-31f0-11ed-9e27-c03c59ad2248";
        // 定义一个Map集合，存储相关的流程变量信息
        Map<String,Object> map = new HashMap<>();
        map.put("user1","demo");
        // 通过 RuntimeService 启动一个流程实例，同时绑定了对应的流程变量信息
        runtimeService.startProcessInstanceById(processInstanceId,map);
    }
```

通过后台查看数据我们可以发现`act_ru_task`中有了一条`人事审批`的任务，而且对于的审批人就是`demo`也就是我们给对应的流程变量的赋值

![image-20220912004819089](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912004819089.png)

而对应的流程变量信息存储在`act_ru_variable`中。

![image-20220912004949936](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912004949936.png)



## 3.方法表达式

&emsp;&emsp;**方法表达式 Method expression**: 调用一个方法，可以带或不带参数。**当调用不带参数的方法时，要确保在方法名后添加空括号（以避免与值表达式混淆）。**传递的参数可以是字面值(literal value)，也可以是表达式，它们会被自动解析。例如： boge3306

```java
${printer.print()}
${myBean.getAssignee()}
${myBean.addNewOrder('orderName')}
${myBean.doSomething(myVar, execution)}
```

&emsp;&emsp;myBean是Spring容器中的个Bean对象，表示调用的是bean的addNewOrder方法.我们通过案例来演示下。我们先定义对应的Service

```java
@Service
public class MyBean {

    public String getAssignee(){
        System.out.println("getAssignee 方法执行了....");
        return "demo";
    }
}
```

&emsp;&emsp;然后我们在对应的流程图中来定义。

![image-20220912005546442](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912005546442.png)



&emsp;&emsp;然后通过部署启动操作来看看。

![image-20220912010330601](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912010330601.png)

![image-20220912010405626](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912010405626.png)

&emsp;&emsp;通过这块的演示，我们就可以基于我们的外部条件来结合我们的逻辑实现动态的来指定对应的处理人。



## 4.监听器配置

&emsp;&emsp;可以使用监听器来完成很多Camunda的流程业务。我们在此处使用监听器来完成负责人的指定，那么我们在流程设计的时候就不需要指定assignee。创建自定义监听器：

```java
/**
 * 自定义的一个 Task 监听器
 * 我们需要在监听器中完成 处理人的动态指派
 */
public class MyFirstTaskListener implements TaskListener {
    @Override
    public void notify(DelegateTask delegateTask) {
        System.out.println("MyFirstTaskListener --- > 执行了");
        // 针对 是创建Task节点的事件
        if(EVENTNAME_CREATE.equals(delegateTask.getEventName())){
            // 指派对应的处理人
            delegateTask.setAssignee("demo");
        }
    }
}
```

&emsp;&emsp;然后在流程图中绑定对应的监听器

![image-20220912011453664](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912011453664.png)

&emsp;&emsp;然后我们部署和启动流程后，可以看到对应的触发效果

![image-20220912011644311](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912011644311.png)

表结构中也可以看到相关的信息

![image-20220912011735815](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220912011735815.png)

说明这块的处理是完全可行的。



# 三、流程变量

&emsp;&emsp;流程变量可以用将数据添加到流程的运行时状态中，或者更具体地说，变量作用域中。改变实体的各种API可以用来更新这些附加的变量。一般来说，一个变量由一个名称和一个值组成。名称用于在整个流程中识别变量。例如，如果一个活动（activity）设置了一个名为 *var* 的变量，那么后续活动中可以通过使用这个名称来访问它。变量的值是一个 Java 对象。

## 1.运行时变量

&emsp;&emsp;流程实例运行时的变量，存入act_ru_variable表中。在流程实例运行结束时，此实例的变量在表中删除。在流程实例创建及启动时，可设置流程变量。所有的`startProcessInstanceXXX`方法都有一个可选参数用于设置变量。例如，`RuntimeService`中

```java
ProcessInstance startProcessInstanceById(String processDefinitionId, Map<String, Object> variables);
```

&emsp;&emsp;也可以在流程执行中加入变量。例如，(*RuntimeService*):

```java
    void setVariable(String executionId, String variableName, Object value);
    void setVariableLocal(String executionId, String variableName, Object value);
    void setVariables(String executionId, Map<String, ? extends Object> variables);
    void setVariablesLocal(String executionId, Map<String, ? extends Object> variables);
```

&emsp;&emsp;读取变量方法:

```java
    Map<String, Object> getVariables(String executionId);
    Map<String, Object> getVariablesLocal(String executionId);
    Map<String, Object> getVariables(String executionId, Collection<String> variableNames);
    Map<String, Object> getVariablesLocal(String executionId, Collection<String> variableNames);
    Object getVariable(String executionId, String variableName);
    <T> T getVariable(String executionId, String variableName, Class<T> variableClass);
```

**注意**：由于流程实例结束时，对应在运行时表的数据跟着被删除。所以查询一个已经完结流程实例的变量，只能在历史变量表中查找。

&emsp;&emsp;当然运行时变量我们也可以根据对应的作用域把他分为`全局变量`和`局部变量`.

### 1.1 全局变量

&emsp;&emsp;流程变量的默认作用域是流程实例。当一个流程变量的作用域为流程实例时，可以称为 global 变量

注意：如：    Global变量：userId（变量名）、zhangsan（变量值）

&emsp;&emsp;global 变量中变量名不允许重复，设置相同名称的变量，后设置的值会覆盖前设置的变量值。

案例：

定义监听器

```java
public class MySecondListener implements TaskListener {

    @Override
    public void notify(DelegateTask delegateTask) {
        // 获取所有的流程变量
        Map<String, Object> variables = delegateTask.getVariables();
        Set<String> keys = variables.keySet();
        for (String key : keys) {
            Object obj = variables.get(key);
            System.out.println(key + " = " + obj);
            if(obj instanceof  String){
              // 修改 流程变量的信息
              // variables.put(key,obj + ":boge3306"); 直接修改Map中的数据 达不到修改流程变量的效果
              delegateTask.setVariable(key,obj + ":boge3306");
            }
        }
    }
}
```

设计流程

![image-20220913011512486](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220913011512486.png)



案例代码：

```java
@SpringBootTest(classes = Application.class)
public class AssigneeTaskTest {

    @Autowired
    RepositoryService repositoryService;

    @Autowired
    RuntimeService runtimeService;


    @Autowired
    TaskService taskService;

    /**
     * 部署流程
     */
    @Test
    public void deployFlow(){
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程-监听器")
                .addClasspathResource("flow/04-任务分配-监听器分配.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void startFlow(){
        String processDefId = "Process_08kf7mp:1:d154c0f5-326d-11ed-841a-c03c59ad2248";
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefId);
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }

    /**
     * 启动流程实例，绑定对应的流程变量
     */
    @Test
    public void startFlowVariables(){
        String processDefId = "Process_1t425hs:1:539f152d-326a-11ed-ab57-c03c59ad2248";
        // 声明一个 Map 集合
        Map<String,Object> map = new HashMap<>();
        map.put("user1","demo");
        // 启动流程实例，同时绑定对应的流程变量信息
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefId,map);

        System.out.println("processInstance.getId() = " + processInstance.getId());
    }

    /**
     * 流程审批
     */
    @Test
    public void completeTask(){
        taskService.complete("29a0614c-3267-11ed-a030-c03c59ad2248");
    }

    /**
     * 流程审批
     *    同样需要绑定对应的流程变量的值
     */
    @Test
    public void completeTaskVariables(){
        // 声明一个 Map 集合
        Map<String,Object> map = new HashMap<>();
        map.put("user2","demo");
        taskService.complete("dd013840-326a-11ed-b589-c03c59ad2248",map);
    }
}
```





### 1.2 局部变量

&emsp;&emsp;任务和执行实例仅仅是针对一个任务和一个执行实例范围，范围没有流程实例大， 称为 local 变量。

&emsp;&emsp;Local 变量由于在不同的任务或不同的执行实例中，作用域互不影响，变量名可以相同没有影响。Local 变量名也可以和 global 变量名相同，没有影响。 

我们通过RuntimeService 设置的Local变量绑定的是 executionId。在该流程中有效

![image-20220913120555429](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220913120555429.png)

```java
    /**
     * 设置Local流程变量
     */
    @Test
    public void setVariableLocal(){
        String executionId = "368f9fdc-3317-11ed-871d-c03c59ad2248";
        runtimeService.setVariableLocal(executionId,"orderId","10006");
        runtimeService.setVariableLocal(executionId,"user2","波波烤鸭1");
    }
```



我们通过TaskService设置的Local变量的作用域是和TaskId绑定的，作用域就在这一个Task生命周期中。

```java
    @Test
    public void taskLocalVariables(){
        taskService.setVariableLocal("7eb1b34b-3318-11ed-b9e6-c03c59ad2248","user2","波哥66666");
    }
```





## 2.历史变量

&emsp;&emsp;历史变量，存入`act_hi_varinst`表中。在流程启动时，流程变量会同时存入历史变量表中；在流程结束时，历史表中的变量仍然存在。可理解为“永久代”的流程变量。

&emsp;&emsp;获取已完成的、id为’XXX’的流程实例中，所有的`HistoricVariableInstances`（历史变量实例），并以变量名排序。

```java
historyService.createHistoricVariableInstanceQuery()
    .processInstanceId("XXX")
    .orderByVariableName.desc()
    .list();
```







# 四、身份服务

&emsp;&emsp;在流程定义中在任务结点的 assignee 固定设置任务负责人，在流程定义时将参与者固定设置在.bpmn 文件中，如果临时任务负责人变更则需要修改流程定义，系统可扩展性差。针对这种情况可以给任务设置多个候选人或者候选人组，可以从候选人中选择参与者来完成任务。

身份服务是对各种用户/组库的API抽象。其基本实体是：

- User: 使用不同ID区分的不同用户
- Group: 使用不同ID区分的不同组
- Membership: 组与用户之间的关系
- Tenant: 使用不同ID区分的不同租户
- Tenant Membership: 租户与 用户/组 之间的关系

## 1.候选人

### 1.1 绘制流程图

&emsp;&emsp;首先绘制一个如下的基本流程图。然后我们分别来指派处理人。

![image-20220917101906123](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917101906123.png)



&emsp;&emsp;人事审批这块我们可以直接来指定多个候选人来处理。`demo,zhang,lisi`

![image-20220917102115513](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917102115513.png)

&emsp;&emsp;在总经理审批的位置我们在设计的时候不太清楚会是谁来审批，所以通过值表达式来处理。

![image-20220917102211814](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917102211814.png)

&emsp;&emsp;设计完成后对应的xml中的数据为：

![image-20220917102259478](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917102259478.png)



### 1.2 部署和启动流程

&emsp;&emsp;创建了流程图后我们就可以直接来部署该流程。

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("候选人案例")
                .addClasspathResource("flow/候选人案例.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```

&emsp;&emsp;接着就可以直接来启动该流程了。

```java
    /**
     * 启动流程实例
     */
    @Test
    public void startFlow(){
        String processInstanceId = "Process_05vjqic:1:cca1e181-362e-11ed-b8fc-c03c59ad2248";
        runtimeService.startProcessInstanceById(processInstanceId);
    }
```

&emsp;&emsp;启动完成流程后我们进入到`act_ru_task`中可以发现我们创建的流程任务信息，但是`处理人`字段还是空的。

![image-20220917102625541](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917102625541.png)



![image-20220917102655865](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917102655865.png)



注意：相关的候选人的信息存储在了`act_ru_identitylink`表中。

![image-20220917104657296](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917104657296.png)

对应的查询操作如下：

```java
    @Test
    public void getTaskCandidate(){
        List<IdentityLink> identityLinksForTask = taskService.getIdentityLinksForTask("023f0279-362f-11ed-8d8a-c03c59ad2248");
        for (IdentityLink identityLink : identityLinksForTask) {
            System.out.println(identityLink.getTaskId());
            System.out.println(identityLink.getProcessDefId());
            System.out.println(identityLink.getUserId());
        }
    }
```





### 1.3 任务的拾取

&emsp;&emsp;候选要操作我们需要通过`拾取`的行为把`候选人`转换为`处理人`.那么候选人登录后需要能查询出来他可以`拾取`的任务。在camunda的web应用中我们可以看到这样的操作。`demo`账号登录。

![image-20220917103124214](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917103124214.png)

&emsp;&emsp;在代码上的实现，先来看查询操作。

```java
    /**
     * 根据登录的用户查询对应的可以拾取的任务
     */
    @Test
    public void queryTaskCandidate(){

        List<Task> list = taskService.createTaskQuery()
                .processInstanceId("023da2e6-362f-11ed-8d8a-c03c59ad2248")
                .taskCandidateUser("demo")
                .list();
        for (Task task : list) {
            System.out.println("task.getId() = " + task.getId());
            System.out.println("task.getName() = " + task.getName());
        }
    }

    /**
     * 查询当前任务所有的候选人
     */
    @Test
    public void getTaskCandidate(){
        String taskId = "52b2642a-36fa-11ed-bde4-c03c59ad2248";
        List<IdentityLink> linksForTask = taskService.getIdentityLinksForTask(taskId);
        if(linksForTask != null && linksForTask.size() > 0){
            for (IdentityLink identityLink : linksForTask) {
                System.out.println(identityLink.getUserId());
            }
        }
    }
```

&emsp;&emsp;然后我们就可以在上面的基础上来做`拾取`的操作了。

```java
    /**
     * 根据登录的用户查询对应的可以拾取的任务
     */
    @Test
    public void claimTaskCandidate(){

        List<Task> list = taskService.createTaskQuery()
                .processInstanceId("023da2e6-362f-11ed-8d8a-c03c59ad2248")
                .taskCandidateUser("demo")
                .list();
        for (Task task : list) {
            taskService.claim(task.getId(),"demo");
        }
    }
```

&emsp;&emsp;进入到表结构中你会发现这条Task记录的处理人被指派为了`demo`,而且在Web端可以看到可以审批了。

![image-20220917103743685](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917103743685.png)



注意：这时`demo`拾取了任务之后其他的用户就不能再拾取了，查询的时候也查询不到了。





### 1.4.任务的归还

&emsp;&emsp;拾取任务后如果不想操作那么可以归还任务。

```java
    /**
     * 退还任务
     *    一个候选人拾取了这个任务之后其他的用户就没有办法拾取这个任务了
     *    所以如果一个用户拾取了任务之后又不想处理了，那么可以退还
     */
    @Test
    public void unclaimTaskCandidate(){
        Task task = taskService.createTaskQuery()
                .processInstanceId("023da2e6-362f-11ed-8d8a-c03c59ad2248")
                .taskAssignee("demo")
                .singleResult();
        if(task != null){
            // 归还相关的任务  置空即可
            taskService.setAssignee(task.getId(),null);
            System.out.println("归还拾取成功");
        }
    }
```



### 1.5 任务的交接

&emsp;&emsp;拾取任务后如果不想操作也不想归还可以直接交接给另外一个人来处理.

```java
    @Test
    public void taskCandidate(){
        Task task = taskService.createTaskQuery()
                .processInstanceId("023da2e6-362f-11ed-8d8a-c03c59ad2248")
                .taskAssignee("demo")
                .singleResult();
        if(task != null){
            // 任务交接
            taskService.setAssignee(task.getId(),"zhang");
            System.out.println("任务交接给了zhang");
        }
    }
```



### 1.6 任务完成

&emsp;&emsp;正常的任务处理

```java
    @Test
    public void completeTask1(){
        String taskId = "023f0279-362f-11ed-8d8a-c03c59ad2248";
        // 但是下一个节点的 处理人是值表达式 我们需要赋值
        Map<String,Object> map = new HashMap<>();
        map.put("user1","demo");
        map.put("user2","zhangsan");
        map.put("user3","wangwu");
        taskService.complete(taskId,map);
    }
```

&emsp;&emsp;当然我们通过值表达式来处理的候选人操作。在`act_ru_identitylink`表中同样有相关记录，我们需要结合流程变量表来处理了。但是处理的API和上面是一样的。



## 2. 候选人组

&emsp;&emsp;当候选人很多的情况下，我们可以分组来处理。先创建组，然后把用户分配到这个组中。



### 2.1 管理用户和组

#### 2.1.1 用户管理

&emsp;&emsp;我们需要先单独维护用户信息。后台对应的表结构是`ACT_ID_USER`.

```java
    /**
     * 维护用户
     */
    @Test
    public void createUser(){
        User user = identityService.newUser("zhang");
        user.setFirstName("张");
        user.setLastName("三");
        user.setEmail("zhangsan@qq.com");
        user.setPassword("123456");
        identityService.saveUser(user);
    }
```

&emsp;&emsp;要更新或者删除用户的话。通过相关API即可完成

```java
    @Test
    public void updateUser(){
        User user = identityService.createUserQuery().userId("zhang").singleResult();
        user.setPassword("123");
        identityService.saveUser(user);
    }
    @Test
    public void deleteUser(){
        identityService.deleteUser("zhang");
    }
```



#### 2.1.2 Group管理

&emsp;&emsp;维护对应的Group信息，后台对应的表结构是`ACT_ID_GROUP`

```java
    /**
     * 创建用户组
     */
    @Test
    public void createGroup(){
        // 创建Group对象并指定相关的信息
        Group group = identityService.newGroup("group1");
        group.setName("开发部");
        group.setType("type1");
        // 创建Group对应的表结构数据
        identityService.saveGroup(group);
    }
```

更新和删除参考上面的`用户管理`



#### 2.1.3 用户分配组

&emsp;&emsp;用户和组是一个多对多的关联关联，我们需要做相关的分配，后台对应的表结构是`ACT_ID_MEMBERSHIP`

```java
    /**
     * 将用户分配给对应的Group
     */
    @Test
    public void userGroup(){
        // 根据组的编号找到对应的Group对象
        Group group = identityService.createGroupQuery().groupId("group1").singleResult();
        // 创建 MemberShip 建立用户和组的关系
        identityService.createMembership("zhang",group.getId());
    }
```



### 2.2 候选人组应用

&emsp;&emsp;搞清楚了用户和用户组的关系后我们就可以来使用候选人组的应用了

#### 2.2.1 创建流程图

&emsp;&emsp;创建一个简单的请假流程，处理人通过候选人组的方式来处理。

![image-20220917201151251](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917201151251.png)

#### 2.2.2 流程的部署运行

&emsp;&emsp;然后我们把流程部署和运行。

```java
 /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("候选人组案例")
                .addClasspathResource("flow/候选人组.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void startFlow(){
        String processInstanceId = "Process_1gvo8so:1:3e253edb-3682-11ed-a1ff-c03c59ad2248";
        runtimeService.startProcessInstanceById(processInstanceId);
    }
```

部署成功后我们可以在`act_ru_identitylink`中看到对应的记录。

![image-20220917201747129](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917201747129.png)



#### 2.2.3 任务的拾取和完成

&emsp;&emsp;然后完成任务的查询拾取和处理操作。逻辑是根据当前的登录`用户`找到对应的`组`，然后根据`组`找到对应的任务信息。

```java
   /**
     * 根据登录的用户查询对应的可以拾取的任务
     *
     */
    @Test
    public void queryTaskCandidateGroup(){
        // 当前用户所在的组
        Group group = identityService.createGroupQuery().groupMember("zhang").singleResult();
        List<Task> list = taskService.createTaskQuery()
                .processInstanceId("711d5726-3682-11ed-8b9b-c03c59ad2248")
                .taskCandidateGroup(group.getId())
                .list();
        for (Task task : list) {
            System.out.println("task.getId() = " + task.getId());
            System.out.println("task.getName() = " + task.getName());
        }
    }

    /**
     * 拾取任务
     *    一个候选人拾取了这个任务之后其他的用户就没有办法拾取这个任务了
     *    所以如果一个用户拾取了任务之后又不想处理了，那么可以退还
     */
    @Test
    public void claimTaskCandidate1(){
        String userId = "zhang";
        // 当前用户所在的组
        Group group = identityService.createGroupQuery().groupMember(userId).singleResult();
        Task task = taskService.createTaskQuery()
                .processInstanceId("711d5726-3682-11ed-8b9b-c03c59ad2248")
                .taskCandidateGroup(group.getId())
                .singleResult();
        if(task != null) {
            // 任务拾取
            taskService.claim(task.getId(),userId);
            System.out.println("任务拾取成功");
        }
    }
```

&emsp;&emsp;拾取后的操作和前面是一样的，就没必要赘述。当然我们在定义流程的时候也可以通过值表达式来处理，我们需要注意赋值即可。



## 3.租户

&emsp;&emsp;*多租户* 是指一个单一的Camunda应用需要为多个的租户服务的情况。对于每个租户来说，应该有某些隔离的保证。例如，一个租户的流程实例不应干扰另一个租户的流程实例。

&emsp;&emsp;多租户可以通过两种不同的方式实现。一种是使用每个租户一个流程引擎。另一种方式是只使用一个流程引擎，并将数据与租户标识符相关联。这两种方式在数据隔离程度、维护工作和可扩展性方面各有不同。两种方式的组合也是可能的。

&emsp;&emsp;多租户可以使用租户标识符（即tenant-ids）的流程引擎来实现。所有租户的数据都存储在一个表中（同一数据库和表结构）。通过存储在列中的租户标识符来提供隔离。

![image-20220917203832905](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220917203832905.png)



### 3.1 租户管理

&emsp;&emsp;租户对应于`act_id.tenant`表结构中的数据。我们可以先来维护租户的相关信息。创建`长沙分公司`的租户信息

```java
    /**
     * 创建租户
     */
    @Test
    public void createTenant(){
        Tenant tenant = identityService.newTenant("cs");
        tenant.setName("长沙分公司");
        identityService.saveTenant(tenant);
    }
```

![image-20220918220534584](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220918220534584.png)



&emsp;&emsp;当然参考前面的删除和更新操作也可以非常轻松的完成`租户`相关信息的处理。然后我们来看下租户和用户和组的关系。

```java
    /**
     * 建立 租户 和 组的关系
     * 当然也可以建立 租户和用户的关系。只是这种比较少用
     */
    @Test
    public void createTenantAndGroupShip(){
        identityService.createTenantGroupMembership("cs","group1");
    }
```

![image-20220918220924950](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220918220924950.png)





### 3.2 部署操作

&emsp;&emsp;我们在部署流程的时候可以指定对应的租户编号。如果不指定租户编号，说明该流程是属于所有租户的。

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("候选人案例-租户")
                .tenantId("tenant1")
                .addClasspathResource("flow/候选人案例.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```



### 3.3 查看部署流程

&emsp;&emsp;设置了租户编号后，我们要做相关的查询，可以通过如下的API来实现

```java
    /**
     * 基于租户 查询相关的部署流程
     */
    @Test
    public void getByTenantId(){
        List<Deployment> list = repositoryService.createDeploymentQuery()
                .tenantIdIn("cs")
                .list();
        for (Deployment deployment : list) {
            System.out.println("deployment.getId() = " + deployment.getId());
            System.out.println("deployment.getName() = " + deployment.getName());
            System.out.println("------------");
        }
    }

    
```

通过调用`withoutTenantId()`来查询不属于任何租户的部署。

```java
@Test
    public void getByWithoutTenantId(){
        List<Deployment> list = repositoryService.createDeploymentQuery()
                .withoutTenantId() // 查询出所有不属于任何tenantId的记录
                .list();
        for (Deployment deployment : list) {
            System.out.println("deployment.getId() = " + deployment.getId());
            System.out.println("deployment.getName() = " + deployment.getName());
            System.out.println("------------");
        }
    }
```



也可以通过调用`includeDeploymentsWithoutTenantId()`来查询属于特定租户或不属于租户的部署。

```
@Test
public void getByIncludTenantId(){
    List<Deployment> list = repositoryService.createDeploymentQuery()
            .tenantIdIn("cs")
            .includeDeploymentsWithoutTenantId() // 查询出 tenant1 和 不属于 租户的记录
            .list();
    for (Deployment deployment : list) {
        System.out.println("deployment.getId() = " + deployment.getId());
        System.out.println("deployment.getName() = " + deployment.getName());
        System.out.println("------------");
    }
}
```

&emsp;&emsp;与 “部署查询” 类似，定义查询允许通过一个或多个租户和不属于任何租户的定义进行过滤。

```java
List<ProcessDefinition> processDefinitions = repositoryService
  .createProcessDefinitionQuery()
  .tenantIdIn("cs")
  .includeProcessDefinitionsWithoutTenantId();
  .list();
```





### 3.4 启动流程实例

&emsp;&emsp;通过key创建一个为多租户部署的流程定义的实例，必须在`ProcessInstantiationBuilder` 中传递租户标识符 。

```java
    /**
     * 租户 启动一个流程实例
     */
    @Test
    public void startFlow(){
        String processKey = "Process_05vjqic";
        ProcessInstance processInstance = runtimeService
                .createProcessInstanceByKey(processKey)
                .processDefinitionTenantId("cs")
                .execute();
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }

    @Test
    public void startFlow1(){
        String processId = "Process_05vjqic:1:a6b23794-3767-11ed-a4df-c03c59ad2248";
        ProcessInstance processInstance = runtimeService
                .createProcessInstanceById(processId)
                .execute();
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }
```

&emsp;&emsp;启动后流程后，在创建的Task记录中我们可以看到对应的`租户`信息

![image-20220918234504559](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220918234504559.png)

&emsp;&emsp;因为我们在流程设计的时候就指定了第一个节点的候选人是`group1`,所以在`act_ru_identitylink`表中可以看到相关的记录。

![image-20220918234618891](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220918234618891.png)



### 3.5 任务拾取

&emsp;&emsp;接下来就可以对任务做`拾取`的操作了

```java
   /**
     * 根据当前登录用户 查询到需要拾取的任务
     */
    @Test
    public void claimTask(){
        // 根据登录用户查询到对应的Group
        List<Group> groups = identityService.createGroupQuery().groupMember("demo").list();
        if(groups != null && groups.size() > 0){
            for (Group group : groups) {
                // 根据 group 找到对应的 租户
                List<Tenant> tenants = identityService.createTenantQuery().groupMember(group.getId()).list();
                List<String> tenantStrings = new ArrayList<>();
                if(tenants != null && tenants.size() > 0){
                    tenantStrings = tenants.stream().map((item)->{
                        return item.getId();
                    }).collect(Collectors.toList());

                    String[] ss = new String[tenantStrings.size()];
                    tenantStrings.toArray(ss);
                    List<Task> list = taskService.createTaskQuery()
                            .tenantIdIn(ss)
                            .list();
                    if(list != null && list.size() > 0){
                        for (Task task : list) {
                            System.out.println("task.getId() = " + task.getId());
                        }
                    }
                }

            }
        }

    }
```

可以查询到对应的拾取就比较简单了

```java
    @Test
    public void claimTask(){
        // 根据登录用户查询到对应的Group
        List<Group> groups = identityService.createGroupQuery().groupMember("demo").list();
        if(groups != null && groups.size() > 0){
            for (Group group : groups) {
                // 根据 group 找到对应的 租户
                List<Tenant> tenants = identityService.createTenantQuery().groupMember(group.getId()).list();
                List<String> tenantStrings = new ArrayList<>();
                if(tenants != null && tenants.size() > 0){
                    tenantStrings = tenants.stream().map((item)->{
                        return item.getId();
                    }).collect(Collectors.toList());

                    String[] ss = new String[tenantStrings.size()];
                    tenantStrings.toArray(ss);
                    List<Task> list = taskService.createTaskQuery()
                            .tenantIdIn(ss)
                            .list();
                    if(list != null && list.size() > 0){
                        for (Task task : list) {
                            taskService.claim(task.getId(),"demo");
                        }
                    }
                }

            }
        }

    }

```

![image-20220919003450202](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220919003450202.png)

能够拾取成功，那么后面的操作就是任务的审批。接下来的操作就和前面是一样的了。不再赘述~







# 五、网关篇

&emsp;&emsp;网关用来控制流程的流向

## 1. 排他网关

&emsp;&emsp;`排他网关`（exclusive gateway）（也叫*异或网关 XOR gateway*，或者更专业的，*基于数据的排他网关 exclusive data-based gateway*），用于对流程中的**决策**建模。当执行到达这个网关时，会按照所有出口顺序流定义的顺序对它们进行计算。选择第一个条件计算为true的顺序流（当没有设置条件时，认为顺序流为*true*）继续流程。

绘制流程图：

![image-20220922002931276](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922002931276.png)



对应的XML文件

![image-20220922003022802](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922003022802.png)

流程演示

```java
   /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("排他网关")
                .addClasspathResource("flow/排他网关.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }


    /**
     * 通过流程定义Id 启动
     */
    @Test
    public void startFlow(){
        String processId = "Process_0eykic0:1:efadfc92-39c9-11ed-8f13-c03c59ad2248";
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processId);
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }

    /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        Map<String,Object> map = new HashMap<>();
        map.put("day",4);
        taskService.complete("21a323ee-39ca-11ed-8b49-c03c59ad2248",map);
    }
```

传递的是`day=4`会走中间的路线。

![image-20220922003119262](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922003119262.png)

## 2. 并行网关

&emsp;&emsp;并行网关允许将流程分成多条分支，也可以把多条分支汇聚到一起，并行网关的功能是基于进入和外出顺序流的：

* fork分支：并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。

* join汇聚： 所有到达并行网关，在此等待的进入分支， 直到所有进入顺序流的分支都到达以后， 流程就会通过汇聚网关。

&emsp;&emsp;注意，如果同一个并行网关有多个进入和多个外出顺序流， 它就同时具有分支和汇聚功能。 这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支。

**与其他网关的主要区别是，并行网关不会解析条件。** **即使顺序流中定义了条件，也会被忽略。**

![image-20220922003323785](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922003323785.png)

## 3.包含网关

&emsp;&emsp;包含网关可以看做是排他网关和并行网关的结合体。 和排他网关一样，你可以在外出顺序流上定义条件，包含网关会解析它们。 但是主要的区别是包含网关可以选择多于一条顺序流，这和并行网关一样。

包含网关的功能是基于进入和外出顺序流的：

* 分支： 所有外出顺序流的条件都会被解析，结果为true的顺序流会以并行方式继续执行， 会为每个顺序流创建一个分支。

* 汇聚：所有并行分支到达包含网关，会进入等待状态， 直到每个包含流程token的进入顺序流的分支都到达。 这是与并行网关的最大不同。换句话说，包含网关只会等待被选中执行了的进入顺序流。 在汇聚之后，流程会穿过包含网关继续执行。



![image-20220922005111023](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922005111023.png)





当输入`day=4`时，走了第二和第三条路线

![image-20220922005536912](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220922005536912.png)





# 六、事件篇

&emsp;&emsp;`事件`（event）通常用于为流程生命周期中发生的事情建模。事件总是图形化为圆圈。在BPMN 2.0中，有两种主要的事件分类：*捕获（catching）*与*抛出（throwing）*事件。

- **捕获:** 当流程执行到达这个事件时，会等待直到触发器动作。触发器的类型由其中的图标，或者说XML中的类型声明而定义。捕获事件与抛出事件显示上的区别，是其内部的图标没有填充（即是白色的）。
- **抛出:** 当流程执行到达这个事件时，会触发一个触发器。触发器的类型，由其中的图标，或者说XML中的类型声明而定义。抛出事件与捕获事件显示上的区别，是其内部的图标填充为黑色。

## 1. 定时器事件

&emsp;&emsp;定时触发的相关事件，包括定时器启动事件，定时器捕获中间件事件，定时器边界事件

### 1.1 定时器启动事件

&emsp;&emsp;定时器启动事件（timer start event）在指定时间创建流程实例。在流程只需要启动一次，或者流程需要在特定的时间间隔重复启动时，都可以使用。

***请注意**：*子流程不能有定时器启动事件。

***请注意**：*定时器启动事件，在流程部署的同时就开始计时。不需要调用startProcessInstanceByXXX就会在时间启动。调用startProcessInstanceByXXX时会在定时启动之外额外启动一个流程。

***请注意**：*当部署带有定时器启动事件的流程的更新版本时，上一版本的定时器作业会被移除。这是因为通常并不希望旧版本的流程仍然自动启动新的流程实例。

定时器启动事件，用其中有一个钟表图标的圆圈来表示。我们通过具体案例来介绍

![image-20220926104055183](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220926104055183.png)

部署流程后等待到具体的时间，我们查看任务即可

![image-20220926104152240](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220926104152240.png)



时间定义这块使用的是  https://en.wikipedia.org/wiki/ISO_8601#Dates  ISO 8601 格式

上面我们是通过指定固定时间来启动的，我们也可以通过`duraction`间隔时间来处理。

![image-20220926105514506](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220926105514506.png)



通过间隔的方式来启动。

timeCycle：指定重复周期，可用于周期性启动流程，或者为超期用户任务多次发送提醒,这个元素可以使用两种格式

* 第一种是按照[ISO 8601](http://en.wikipedia.org/wiki/ISO_8601#Repeating_intervals)标准定义的循环时间周期。例如（三次重复间隔，每次间隔为10小时）：R3/PT10H
* 也可以使用*timeCycle*的可选属性*endDate*，或者像这样直接写在时间表达式的结尾：`R3/PT10H/${EndDate}`。 当到达endDate时，应用会停止，并为该任务创建其他作业
* 也可以通过cron表达式来处理



案例：重复时间设置为 R3PT30S 重复3次，间隔30描述，自动任务绑定的是JavaDelegate

```java
public class MyJavaDelegate implements JavaDelegate {
    @Override
    public void execute(DelegateExecution execution) throws Exception {
        System.out.println("MyJavaDelegate:执行了。。。"+ LocalDateTime.now().toString());
    }
}
```

![image-20220926110231335](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220926110231335.png)

在控制台可以看到对应的效果

![image-20220926110317226](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220926110317226.png)

也可以指定结束时间

```xml
      <timerEventDefinition>
        <timeCycle>R3/PT30S/2022-03-28T21:46:11+00:00</timeCycle>
      </timerEventDefinition>
```



此外还可以通过cron表达式来处理：

```xml
0 0/5 * * * ?
```

每隔5秒启动

### 1.2 定时器中间事件

&emsp;&emsp;在我们具体的流程处理中，A节点处理完成后，定时触发B节点的处理。

![image-20220927002456358](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927002456358.png)

### 1.3 定时器边界事件

&emsp;&emsp;人工任务1如果在定义的`2022-09-27T23:36:14`这个时间之前还没有处理，那么就会触发定时边界事件，从而从人工任务3.

![image-20220927003701033](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927003701033.png)





## 2.消息事件

### 2.1 启动事件

&emsp;&emsp;消息启动事件，也就是我们通过接收到某些消息后来启动流程实例，比如接收到了一封邮件，一条短信等，具体通过案例来讲解

![image-20220927004852502](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927004852502.png)



&emsp;&emsp;启动流程实例可以通过普通的方式来触发，当然也可以通过对应的API来处理

```java
    /**
     * 消息启动事件
     */
    @Test
    public void startMessageFlow(){
        runtimeService.startProcessInstanceByMessage("firstMessage");
    }
```



### 2.2  中间事件

&emsp;&emsp;消息中间事件就是在流程运作中需要消息来触发的场景，案例演示，`自动流程1`处理完成后，需要接收特定的消息之后才能进入到`自动流程2`

![image-20220927005944099](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927005944099.png)

&emsp;&emsp;正常的流程部署，启动和审批后，我们需要发送对应的消息来触发这个中间事件。

```java
    /**
     * 发送消息
     */
    @Test
    public void sendMsg(){
        runtimeService.messageEventReceived("secondMessage","da2028e9-3dbc-11ed-adb0-c03c59ad2248");
    }
```





### 2.3 边界事件

&emsp;&emsp;消息边界事件，如果在消息触发前还没有，案例演示：

![image-20220927010931908](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927010931908.png)

&emsp;&emsp;然后正常的部署，启动流程，如果在发布对应的消息之前还没有审批`用户任务1`,那当我们发送对应的消息后就会触发对应的消息边界事件。

```java
    /**
     * 发送消息
     */
    @Test
    public void sendMsg(){
        runtimeService.messageEventReceived("thirdMessage","0d2a30e2-3dbf-11ed-9e28-c03c59ad2248");
    }
```





## 3.错误事件

### 3.1 开始事件

&emsp;&emsp;错误启动事件（error start event），可用于触发事件子流程（Event Sub-Process）。**错误启动事件不能用于启动流程实例**。

错误启动事件总是中断。我们通过案例来介绍。

![image-20220927113303497](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113303497.png)

绘制事件子流程要注意：

![image-20220927113345613](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113345613.png)

选择错误启动事件

![image-20220927113423993](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113423993.png)



然后配置流程节点信息

![image-20220927113506045](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113506045.png)

![image-20220927113528120](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113528120.png)

然后在JavaDelegate中抛出对应的异常

```java
public class FirstJavaDelegate implements JavaDelegate {
    @Override
    public void execute(DelegateExecution execution) throws Exception {
        System.out.println("FirstJavaDelegate:执行了" + LocalDateTime.now().toString());
        // 抛出的信息必须对应于error的Code信息
        throw new BpmnError("errorCode01");
    }
}
```

然后正常部署，启动流程。然后我们就可以看到对应的流转了

![image-20220927113652951](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927113652951.png)



### 3.2 边界事件

&emsp;&emsp;当子流程执行中对外抛出了相关的异常，那么我们设置的错误边界事件就能对应的捕获到相关的事件，然后做对应的处理，相关案例如下：

![image-20220927114818638](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927114818638.png)

绘制错误边界流程图的时候需要先绘制中间事件的图标，然后拖拽到子流程的边界，然后修改对应的类型即可，错误边界事件绑定抛出对应的errorCode

![image-20220927114939647](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927114939647.png)

然后对应的抛出的信息

```java
public class SecondJavaDelegate implements JavaDelegate {
    @Override
    public void execute(DelegateExecution execution) throws Exception {
        System.out.println("SecondJavaDelegate:执行了" + LocalDateTime.now().toString());
        // 抛出的信息必须对应于error的Code信息
        throw new BpmnError("errorCode02");
    }
}
```



然后部署启动流程即可看到对应的效果

![image-20220927115037027](https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220927115037027.png)





## 4.信号事件

### 4.1 开始事件

&emsp;&emsp;通过信号来启动流程实例

![image-20220928232328300](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928232328300.png)

部署流程和发送信号来启动流程

```java
    /**
     * 部署流程
     */
    @Test
    public void deployFlow(){

        Deployment deploy = repositoryService.createDeployment()
                .name("事件-信号开始事件")

                .addClasspathResource("flow/23-事件-信号事件-开始事件.bpmn")

                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }

    /**
     * 通过信号来触发启动事件的执行
     */
    @Test
    public void signalRecevied(){
        runtimeService.signalEventReceived("signal01");
    }
```

效果：

![image-20220928232951167](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928232951167.png)

### 4.2 中间捕获事件

&emsp;&emsp;案例如下：当我们启动事件后，会阻塞在这个消息获取中间事件处，等待相关信号后才会继续流转。

![image-20220928233200534](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928233200534.png)

&emsp;&emsp;在这个案例中我们需要部署流程，启动流程，然后`用户任务1`审批后，我们发布相关的信号，然后会进入到`用户任务2`中

```java
   /**
     * 部署流程
     */
    @Test
    public void deployFlow(){

        Deployment deploy = repositoryService.createDeployment()
                .name("事件-信号中间捕获事件")

                .addClasspathResource("flow/24-事件-信号事件-中间捕获事件.bpmn")

                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void startFlow(){
        String processDefId = "Process_002gvxw:1:cfcc56ac-3f42-11ed-a689-c03c59ad2248";

        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefId);
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }

    @Test
    public void signalRecevied(){
        runtimeService.signalEventReceived("signal02");
    }
```

效果：

![image-20220928233556274](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928233556274.png)

### 4.3 中间抛出事件

&emsp;&emsp;信号中间抛出事件也就是在流程执行中的某个节点抛出了对应的信号，然后对应的信号中间捕获事件就会触发，我们通过具体的案例来演示如：

![image-20220928234118517](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928234118517.png)

正常的部署流程，然后做流程的审批，注意`用户任务4`需要先审批，然后处于信号捕获的状态，然后我们`用户任务2`的审批，抛出信号事件，那么对应的捕获事件才能触发，这儿有先后顺序。

![image-20220928235157519](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928235157519.png)

### 4.4 边界事件

&emsp;&emsp;最后来看看信号边界事件，案例如下：

![image-20220928235443496](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928235443496.png)

部署启动流程，如果`用户任务1`审批了，就会进入到`用户任务2`审批，如果`用户任务1`还没审批，然后发送的相关的信号，会被信号边界事件捕获，从而进入到`用户任务3`

```java
    @Test
    public void signalRecevied(){
        runtimeService.signalEventReceived("signal04");
    }
```



![image-20220928235841363](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220928235841363.png)





## 5.结束事件

### 5.1 错误结束事件

&emsp;&emsp;当流程执行到达**错误结束事件（error end event）**时，结束执行的当前分支，并抛出错误。这个错误可以由匹配的错误边界中间事件捕获。如果找不到匹配的错误边界事件，将会抛出异常。通过具体案例来详细讲解：

![image-20220929002020963](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929002020963.png)



部署流程，然后启动流程，`用户任务1`审批

![image-20220929002528362](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929002528362.png)

设置流程变量`flag==1`然后会走`错误结束事件`。会触发对应的边界事件

![image-20220929002622297](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929002622297.png)



### 5.2 中断结束事件

&emsp;&emsp;中断结束事件也称为终止结束事件，主要是对流程进行终止的事件，可以在一个复杂的流程中，如果某方想要提前中断这个流程，可以采用这个事件来处理，可以在并行处理任务中。如果你是在流程实例层处理，整个流程都会被中断，如果是在子流程中使用，那么当前作用和作用域内的所有的内部流程都会被终止。具体还是通过两个案例来给大家介绍：

#### 案例一

&emsp;&emsp;介绍没有子流程的情况下终止流程的场景：

![image-20220929003541120](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929003541120.png)





#### 案例二

&emsp;&emsp;案例二我们来看看有子流程的场景下：

![image-20220929004654376](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929004654376.png)

### 5.3 取消结束事件

&emsp;&emsp;取消结束事件（cancel end event）只能与BPMN事务子流程（BPMN transaction subprocess）一起使用。当到达取消结束事件时，会抛出取消事件，且必须由取消边界事件（cancel boundary event）捕获。取消边界事件将取消事务，并触发补偿（compensation）。

具体通过案例来讲解：



![image-20220929010451477](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929010451477.png)

效果：

![image-20220929010913632](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929010913632.png)



### 5.4 补偿事件

&emsp;&emsp;通过补偿达到控制业务流程的目的就是补偿事件，比如我们正常的买机票的流程下订单购买，然后同时弹出支付流程页面。支付成功后就可以等待出票了，但是如果我们支付失败的话，这时要么重新支付，更换支付方式或者取消预订，这时取消预订我们就可以通过补偿事件来实现，具体的案例如下：

![image-20220929012202047](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929012202047.png)



案例效果：

![image-20220929012149315](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20220929012149315.png)





# 七、多人会签

Multiple Instance

## 1.会签说明

&emsp;&emsp;多实例活动是为业务流程中的某个步骤定义重复的一种方式。在编程概念中，多实例与 `for each` 结构相匹配：它允许对给定集合中的每个项目按顺序或并行地执行某个步骤或甚至一个完整的子流程。

&emsp;&emsp;多实例是一个有额外属性（所谓的 “多实例特性”）的常规活动，它将导致该活动在运行时被多次执行。以下活动可以成为多实例活动。

- Service Task 服务任务
- Send Task 发送任务
- User Task 用户任务
- Business Rule Task 业务规则任务
- Script Task 脚本任务
- Receive Task 接收任务
- Manual Task 手动任务
- (Embedded) Sub-Process （嵌入）子流程
- Call Activity 发起活动
- Transaction Subprocess 事务子流程

网关或事件不能成为多实例。

&emsp;&emsp;如果一个活动是多实例的，这将由活动底部的三条短线表示。三条垂直线表示实例将以**并行**方式执行，而三条水平线表示**顺序**执行。

![image-20221001234837327](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221001234837327.png)





按照规范的要求，每个实例所创建的执行的每个父执行将有以下变量：

- **nrOfInstances**: 实例的总数量
- **nrOfActiveInstances**: 当前活动的，即尚未完成的实例的数量。对于一个连续的多实例，这将永远是1。
- **nrOfCompletedInstances**: 已经完成的实例的数量。

这些值可以通过调用 “execution.getVariable(x) “方法检索。

此外，每个创建的执行将有一个执行本地变量（即对其他执行不可见，也不存储在流程实例级别）。

- **loopCounter**: 表示该特定实例的`for each`循环中的索引

为了使一个活动成为多实例，活动xml元素必须有一个`multiInstanceLoopCharacteristics`子元素。

```xml
<multiInstanceLoopCharacteristics isSequential="false|true">
 ...
</multiInstanceLoopCharacteristics>
```

`isSequential`属性表示该活动的实例是按顺序执行还是并行执行。

## 2.并行会签

### 2.1 绘制流程图

![image-20221002000056981](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002000056981.png)

>loop cardinality：循环基数。可选项。可以直接填整数，表示会签的人数。
>Collection：集合。可选项。会签人数的集合，通常为list，和loop cardinality二选一。
>Element variable：元素变量。选择Collection时必选，为collection集合每次遍历的元素。
>Completion condition：完成条件。可选。比如设置一个人完成后会签结束，那么其他人的代办任务都会消失。

在用户任务节点绑定了一个监听器，监听`create`行为，该监听器我们是通过UEL表达式来实现的，`mulitiInstanceTaskListener`是我们注入到Spring容器中的对象。

![image-20221002000233782](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002000233782.png)

监听器的内容为：

```java
@Component("mulitiInstanceTaskListener")
public class MulitiInstanceTaskListener implements Serializable {
    
    public void completeListener(DelegateExecution execution){
        System.out.println("任务："+execution.getId());
        System.out.println("persons:" + execution.getVariable("persons"));
        System.out.println("person" + execution.getVariable("person"));
        
    }
}
```

然后对应的`Completion condition`我们可以在Java代码中处理

```java
@Component("mulitiInstanceCompleteTask")
public class MulitiInstanceCompleteTask implements Serializable {
    /**
     * 完成任务是需要触发的方法
     * @param execution
     * @return
     *     false 表示会签任务还没有结束
     
     *     true 表示会签任务结束了
     */
    public boolean completeTask(DelegateExecution execution) {
        System.out.println("总的会签任务数量：" + execution.getVariable("nrOfInstances")
                + "当前获取的会签任务数量：" + execution.getVariable("nrOfActiveInstances")
                + " - " + "已经完成的会签任务数量：" + execution.getVariable("nrOfCompletedInstances"));
        //有一个人同意就通过
        Boolean flag = (Boolean) execution.getVariable("flag");
        System.out.println("当前意见："+flag);
        return flag;
    }
}
```

上面的三个变量是Flowable中自带的可用变量

1. nrOfInstances:该会签环节中总共有多少个实例

2. nrOfActiveInstances:当前活动的实例的数量，即还没有完成的实例数量。

3. nrOfCompletedInstances:已经完成的实例的数量



### 2.2 部署流程

&emsp;&emsp;普通的部署流程

```java
    /**
     * 部署流程
     */
    @Test
    public void deployFlow(){

        Deployment deploy = repositoryService.createDeployment()
                .name("多实例")
                .addClasspathResource("flow/diagram_1.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```



### 2.3 启动流程

&emsp;&emsp;在启动流程实例的时候，我们需要设置相关的参数，在流程定义的时候设置的persons在此处我们就需要设置了，设置为Arrays.asList("张三","李四","王五","赵六")，这里设置了4个元素，在流程定义里定义了3个，表示只会循环3次，启动流程后，在Task中可以看到只有3个任务

```java
    @Test
    void startFlow() throws Exception{
        Map<String,Object> map = new HashMap<>();
        // 设置多人会签的数据
        map.put("persons", Arrays.asList("张三","李四","王五","赵六"));
        ProcessInstance processInstance = runtimeService
                .startProcessInstanceById("myProcess:1:ba1518fc-b22d-11ec-9313-c03c59ad2248",map);
    }
```



启动后我们就可以在数据库中看到产生了3个Task

![image-20221002000640899](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002000640899.png)



### 2.4 处理任务

&emsp;&emsp;启动流程后我们发下在Task中产生了3条任务，我们先通过TaskService来完成其中一个任务，设置一个标志flag为false，来控制会签还没有结束，同时Task中另外两个任务还在

```java
    @Test
    void completeTask1(){
        Map<String,Object> map = new HashMap<>();
        map.put("flag",false);
        taskService.complete("71337501-b22e-11ec-a534-c03c59ad2248",map);
    }
```

当任务执行完成时会同步触发会签完成表达式中对象方法。有如下的输出

![image-20221002000853424](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002000853424.png)

同时Task表中的记录还有两条

![image-20221002000931060](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002000931060.png)

然后当我们在完成一个任务，这时设置flag为true，会发现在这个多人处理中，最多3个人处理在第二个人处理后就结束了

```java
    @Test
    void completeTask1(){
        Map<String,Object> map = new HashMap<>();
        map.put("flag",true); // 设置为true 结束多人会签
        taskService.complete("713570d4-b22e-11ec-a534-c03c59ad2248",map);
        System.out.println("complete ....");
    }
```

这时任务节点就结束了~

## 3.串行会签

&emsp;&emsp;串行会签指的是在多实例中，处理人按照对应的顺序来一个个的处理任务。

### 3.1 流程图

![image-20221002001501320](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221002001501320.png)

上面定义的是一个`串行会签`的流程图。循环3次，集合数据在`persons`中，每次循环的数据存储在循环变量`person`中，同时`person`也是对应的处理人的流程变量。



### 3.2 部署和启动流程

&emsp;&emsp;部署就是正常的部署了，不再赘述，启动流程的时候我们需要对集合做赋值。

```java
    @Test
    public void startFlow(){
        Map<String,Object> map = new HashMap<>();
        // 设置多人会签的数据 串行会按照集合的循序来处理
        map.put("persons", Arrays.asList("张三","李四","王五","赵六"));
        String processDefId = "Process_0ltwc4j:1:bd87ed49-41a3-11ed-9ec4-c03c59ad2248";
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefId,map);
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }
```

启动后会创建一个Task，处理人是`张三`。然后处理完成后会创建下一个task，处理人是`李四`按循序来处理了。





# 八、任务回退

&emsp;&emsp;任务回退驳回撤销相关的操作在实际的开发中还是会经常遇到的，我们来看看Camunda中针对这些情况是如何处理的。

## 1.串行的回退

&emsp;&emsp;我们先从最简单的串行流程来分析，案例如下

![image-20221005155134800](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005155134800.png)

&emsp;&emsp;上面的流程就是一个非常简单的串行任务，定义了4个用户任务，指派的处理人分别是`user1`,`user2`,`user3`,`user4`.在流程的执行过程中我们可以通过回退来演示具体的效果。首先来部署流程

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("回退")
                .addClasspathResource("flow/串行回退.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
    /**
     * 通过流程定义Id 启动
     */
    @Test
    public void startFlow(){
        String processId = "Process_1up6ocm:1:d924ab29-43f9-11ed-b4f1-c03c59ad2248";
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processId);
        System.out.println("processInstance.getId() = " + processInstance.getId());
    }
```

然后我们通过user1完成任务，直到user3完成任务，到user4来处理任务。

```java
    /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        taskService.complete("45bf40bf-43fa-11ed-8abe-c03c59ad2248");
    }
```

![image-20221004233745178](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221004233745178.png)

然后我们来完成回退到`user3`

```java
    /**
     * 任务回退
     *
     */
    @Test
    public void rollbackTask(){
        String processInstanceId = "cb7b11bf-4486-11ed-bbb6-c03c59ad2248";
        runtimeService.createProcessInstanceModification(processInstanceId)
                        .startBeforeActivity("Activity_17ubvwq") // 回退到对应的节点前  通过 节点的key 来实现
                       // .cancelActivityInstance(processInstanceId) // 整个任务会取消
                        .cancelAllForActivity("Activity_0jyi1pp") // 取消原来的节点
                        .execute();
        /*String processDefinitionId = "";
        runtimeService.createModification("");*/
    }
```

效果：

![image-20221005155543672](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005155543672.png)

然后在历史表中可以看到对应的走向

![image-20221005155709087](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005155709087.png)

具体过程为：

![image-20221005155909565](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005155909565.png)

然后我们可以从`user3`回退到`user1`

![image-20221005160133670](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005160133670.png)

![image-20221005160200316](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005160200316.png)

要取消当前的任务直接调用`cancelActivityInstance(processInstanceId)`方法即可。

相关方法说明：

| 类别                       | 方法                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 在活动前启动               | 回退到这个节点                                               |
|                            | startBeforeActivity(String activityId)                       |
|                            | startBeforeActivity(String activityId, String ancestorActivityInstanceId) |
| 在活动后启动               | 通过 `startAfterActivity` 在一个活动之后运行，意味着将从活动的下一节点开始执行 |
|                            | startAfterActivity(String activityId)                        |
|                            | startAfterActivity(String activityId, String ancestorActivityInstanceId) |
| 启动一个过渡               | 通过 `startTransition` 启动一个过渡，就意味着在一个给定的序列流上开始执行。当有多个流出的序列流时，这可以与 `startAfterActivity` 一起使用。如果成功，该指令从序列流开始执行流程模型，直到达到等待状态。 |
|                            | startTransition(String transitionId)                         |
|                            | startTransition(String transition, String ancestorActivityInstanceId) |
| 取消一个活动实例           | 通过`cancelActivityInstance`取消一个特定的活动实例。既可以是一个活动实例，如用户任务的实例，也可以是层次结构中更高范围的实例，如子流程的实例 |
|                            | cancelActivityInstance(String activityInstanceId)            |
| 取消一个过渡实例           | 过渡实例表示即将以异步延续的形式进入/离开一个活动的执行流。一个已经创建但尚未执行的异步延续Job被表示为一个过渡实例。这些实例可以通过`cancelTransitionInstance`来取消 |
|                            | cancelTransitionInstance(String activityInstanceId)          |
| 取消一个活动的所有活动实例 | 通过指令 `cancelAllForActivity` 来取消一个特定活动的所有活动和过渡实例。 |
|                            | cancelAllForActivity(String activityId)                      |



## 2.并行的回退

&emsp;&emsp;接下来我们在并行的场景中来看看各种回退的场景。具体案例流程如下：

![image-20221005193633031](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005193633031.png)

&emsp;&emsp;部署后，启动流程实例。运行到如下的节点，然后回退到`用户审批1`

![image-20221005193931957](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005193931957.png)

&emsp;&emsp;然后我们来做回退操作

```java
    @Test
    public void rollbackTask(){
        String processInstanceId = "a1d40268-44a2-11ed-9b63-c03c59ad2248";
        runtimeService.createProcessInstanceModification(processInstanceId)
                        .startBeforeActivity("Activity_0n0ddnj") // 回退到对应的节点前  通过 节点的key 来实现
                       // .cancelActivityInstance(processInstanceId) // 整个任务会取消
                        .cancelAllForActivity("Activity_184rpuo") // 取消原来的节点
                        .cancelAllForActivity("Activity_08w6j91")
                        .execute();
        /*String processDefinitionId = "";
        runtimeService.createModification("");*/
    }
```

&emsp;&emsp;上面我们根据流程`实例回退`同时需要取消掉之前的两个节点。在历史表中也可以看到这个流程

![image-20221005194635852](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005194635852.png)

&emsp;&emsp;在上面例子的基础上我们来实现如下的流程回退操作。

![image-20221005194807783](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005194807783.png)

![image-20221005194904013](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005194904013.png)

具体的回退代码

```java
    @Test
    public void rollbackTask(){
        String processInstanceId = "a1d40268-44a2-11ed-9b63-c03c59ad2248";
        runtimeService.createProcessInstanceModification(processInstanceId)
                        .startBeforeActivity("Activity_184rpuo") // 回退到对应的节点前  通过 节点的key 来实现
                        .startBeforeActivity("Activity_08w6j91")
                        .cancelAllForActivity("Activity_0ma3n4p") // 取消原来的节点
                        .execute();
    }
```

然后查看历史任务同样可以看到具体的实现

![image-20221005195142559](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005195142559.png)



## 3.排他网关回退

&emsp;&emsp;然后我们来看看在有排他网关的情况下，如果我们要调整流程的走向应该要怎么来处理。

![image-20221005200528761](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005200528761.png)

&emsp;&emsp;也就是实际的场景中可能先走到了`用户2`审批，后面发现不合理，就需要回退到`用户1`后面然后再调整流程变量，流转到`用户3`处。

![image-20221005200901486](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005200901486.png)

然后我们来看看具体的回退操作

```java
    @Test
    public void rollbackTask(){
        String processInstanceId = "6a70a33b-44a6-11ed-8557-c03c59ad2248";
        runtimeService.createProcessInstanceModification(processInstanceId)
                        .startBeforeActivity("Gateway_0ah4coh") // 回退到排他网关之前
                        .setVariable("num",-1) // 设置流程变量
                        .cancelAllForActivity("Activity_0x9stxl") // 取消原来的节点
                        .execute();
    }

```

回退到`排他网关`节点，然后设置流程变量`num=-1`然后会走`用户3`的审批。同时取消原来节点的审批

![image-20221005201429581](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005201429581.png)

## 4.子流程回退

&emsp;&emsp;然后我们来看看有子流程的场景下的回退处理

![image-20221005204950474](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005204950474.png)

我们来看看从子流程回退到主流程的操作。

![image-20221005205324729](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005205324729.png)

```java
    @Test
    public void rollbackTask(){
        String processInstanceId = "a3311551-44ac-11ed-97c0-c03c59ad2248";
        runtimeService.createProcessInstanceModification(processInstanceId)
                        .startBeforeActivity("Activity_00kwccq") // 回退到排他网关之前
                        .cancelAllForActivity("Activity_01835pn") // 取消原来的节点
                        .execute();
    }
```

在历史表中查看记录

![image-20221005205659817](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005205659817.png)



## 5.重启/恢复实例

&emsp;&emsp;在流程实例终止后，其历史数据仍然存在，并且可以被访问以重启流程实例，前提是历史级别被设置为FULL。 例如，当流程没有以期望的方式终止时，重启流程实例是有用的。这个API的使用的其他可能情况有：

- 恢复被错误地取消的流程实例的到最后状态
- 由于错误路由导致流程实例终止后，重启流程实例

为了执行这样的操作，流程引擎提供了 *流程实例重启API* `RuntimeService.restartProcessInstances(..)` 。该API允许通过使用流式构建器在一次调用中指定多个实例化指令。

![image-20221005210910364](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005210910364.png)

部署启动流程实例后进入并行网关。然后流程实例被中断了

```java
    @Test
    public void deleteProcessId(){
        String processId = "cfd58b31-44ae-11ed-92f5-c03c59ad2248";
        runtimeService.deleteProcessInstance(processId,"测试删除");
    }
```

![image-20221005211329315](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221005211329315.png)

之后，我们决定恢复该流程实例到最后状态：

```java
    @Test
    public void recoveryProcessInstancess(){
        runtimeService
                .restartProcessInstances("Process_08wyw1g:1:ad8f483f-44ae-11ed-aee8-c03c59ad2248") // 流程定义ID
                .startBeforeActivity("Activity_03vnv63")
                .startBeforeActivity("Activity_0b7mtmx")
                .processInstanceIds("cfd58b31-44ae-11ed-92f5-c03c59ad2248") // 流程实例ID
                .execute();
    }
```

流程实例已经用最后一组变量重启了。然而，在重启的流程实例中，只有全局变量被恢复了。 本地变量还需要调用 `RuntimeService.setVariableLocal(..)` 手动设置。

>从技术上讲，创建的是一个新的流程实例。
>
>**请注意:** 历史流程和重启的流程实例的id是不同的。





# 九、动态表单

&emsp;&emsp;接下来我们看看动态表单的应用，在Camunda中表单分为内置表单和动态表单。

## 1.内置表单

&emsp;&emsp;内置表单就是在绘制流程图的时候同时绘制表单。这种方式其实就是绑定了对应的流程变量，不是太灵活。但还是来讲解下。

![image-20221014160336988](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014160336988.png)

### 1.1 启动流程绑定

&emsp;&emsp;我们先来看下在启动流程的时候就设置相关的表单

![image-20221014160501005](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014160501005.png)

对应的流程图

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:camunda="http://camunda.org/schema/1.0/bpmn" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:modeler="http://camunda.org/schema/modeler/1.0" id="Definitions_1pnd5o3" targetNamespace="http://bpmn.io/schema/bpmn" exporter="Camunda Modeler" exporterVersion="4.12.0" modeler:executionPlatform="Camunda Platform" modeler:executionPlatformVersion="7.15.0">
  <bpmn:process id="Process_13abncz" isExecutable="true">
    <bpmn:startEvent id="StartEvent_1">
      <bpmn:extensionElements>
        <camunda:formData>
          <camunda:formField id="days" label="请假天数" type="long" defaultValue="1" />
          <camunda:formField id="reason" label="请假原因" type="string" />
        </camunda:formData>
      </bpmn:extensionElements>
      <bpmn:outgoing>Flow_09xirvq</bpmn:outgoing>
    </bpmn:startEvent>
    <bpmn:sequenceFlow id="Flow_09xirvq" sourceRef="StartEvent_1" targetRef="Activity_0a81mdi" />
    <bpmn:sequenceFlow id="Flow_1i0viip" sourceRef="Activity_0a81mdi" targetRef="Activity_01qg6se" />
    <bpmn:endEvent id="Event_1mj58zz">
      <bpmn:incoming>Flow_1asp8i3</bpmn:incoming>
    </bpmn:endEvent>
    <bpmn:sequenceFlow id="Flow_1asp8i3" sourceRef="Activity_01qg6se" targetRef="Event_1mj58zz" />
    <bpmn:userTask id="Activity_0a81mdi" name="用户任务1" camunda:assignee="demo">
      <bpmn:incoming>Flow_09xirvq</bpmn:incoming>
      <bpmn:outgoing>Flow_1i0viip</bpmn:outgoing>
    </bpmn:userTask>
    <bpmn:userTask id="Activity_01qg6se" name="用户任务2" camunda:assignee="demo">
      <bpmn:incoming>Flow_1i0viip</bpmn:incoming>
      <bpmn:outgoing>Flow_1asp8i3</bpmn:outgoing>
    </bpmn:userTask>
  </bpmn:process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_1">
    <bpmndi:BPMNPlane id="BPMNPlane_1" bpmnElement="Process_13abncz">
      <bpmndi:BPMNEdge id="Flow_09xirvq_di" bpmnElement="Flow_09xirvq">
        <di:waypoint x="188" y="117" />
        <di:waypoint x="240" y="117" />
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge id="Flow_1i0viip_di" bpmnElement="Flow_1i0viip">
        <di:waypoint x="340" y="117" />
        <di:waypoint x="400" y="117" />
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge id="Flow_1asp8i3_di" bpmnElement="Flow_1asp8i3">
        <di:waypoint x="500" y="117" />
        <di:waypoint x="562" y="117" />
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNShape id="_BPMNShape_StartEvent_2" bpmnElement="StartEvent_1">
        <dc:Bounds x="152" y="99" width="36" height="36" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="Event_1mj58zz_di" bpmnElement="Event_1mj58zz">
        <dc:Bounds x="562" y="99" width="36" height="36" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="Activity_09e81f3_di" bpmnElement="Activity_0a81mdi">
        <dc:Bounds x="240" y="77" width="100" height="80" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="Activity_0ot1277_di" bpmnElement="Activity_01qg6se">
        <dc:Bounds x="400" y="77" width="100" height="80" />
      </bpmndi:BPMNShape>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</bpmn:definitions>

```

可以看到对应的xml中就配置了表单的数据

![image-20221014160601636](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014160601636.png)



部署流程

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("表单01")
                .addClasspathResource("flow/表单-01.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```

部署完流程后我们可以通过对应的API查看流程对应的表单信息

```java
@Test
public void getStartFromData(){
    String processDefId = "Process_13abncz:1:5d39867a-4b6e-11ed-9748-c03c59ad2248";
    // 根据流程定义找到对应的 表单数据。
    StartFormData startFormData = formService
        .getStartFormData(processDefId);
    // 找到对应的表单域的集合
    List<FormField> formFields = startFormData.getFormFields();
    for (FormField formField : formFields) {
        // 获取每个具体的表单域
        System.out.println(formField.getId() + " " + formField.getLabel() + " " + formField.getValue().getValue());
    }
}
```

![image-20221014161737726](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014161737726.png)

启动流程：两种方式

```java
    /**
     * 正常的启动流程
     */
    @Test
    void startFormFlow() throws Exception{
        String processDefId = "Process_13abncz:1:5d39867a-4b6e-11ed-9748-c03c59ad2248";
        Map<String,Object> map = new HashMap<>();
        map.put("days","5");
        map.put("reason","想休息下");
        // 正常启动的时候可以通过流程变量来给表单设置数据，也可以不设置
        ProcessInstance processInstance = runtimeService
                .startProcessInstanceById(processDefId,map);
    }

    /**
     * 通过FormService来启动一个表单流程
     * @throws Exception
     */
    @Test
    void startFormServiceFlow() throws Exception{
        String processDefId = "Process_13abncz:1:5d39867a-4b6e-11ed-9748-c03c59ad2248";
        Map<String,Object> map = new HashMap<>();
        map.put("days",2);
        map.put("reason","出去玩玩");
        ProcessInstance processInstance = formService.submitStartForm(processDefId,map);
    }
```

启动流程后走到了`用户任务1`这个节点

![image-20221014162003762](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014162003762.png)

想要查看对应的表单数据。通过如下方法可以实现

```java
    /**
     * 根据Task编号来查看表单数据
     */
    @Test
    void getTaskFormData(){
        String taskId = "426b11e6-4b85-11ed-b15a-c03c59ad2248";

        String processDefId = "Process_13abncz:1:5d39867a-4b6e-11ed-9748-c03c59ad2248";
        // 根据流程定义找到对应的 表单数据。
        StartFormData startFormData = formService.getStartFormData(processDefId);
        List<FormField> formFields = startFormData.getFormFields();
        for (FormField formField : formFields) {
            // 获取每个具体的表单域
            System.out.println(formField.getId() + " " + formField.getLabel() + " " + formField.getValue().getValue());
            // 根据taskId 找到对应的流程变量的值 也就是 表单的数据
            System.out.println(formField.getId() + " = " + taskService.getVariable(taskId, formField.getId()));
        }
    }
```



![image-20221014162117179](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014162117179.png)

### 1.2 任务节点绑定

&emsp;&emsp;上的方式绑定的表单数据在整个流程实例运作中都可以使用，还有一种方式我们是在具体的任务节点上绑定表单数据。我们来看看。

![image-20221014162658765](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014162658765.png)

流程图中是在 `用户任务01` 这个节点绑定的表单数据了

![image-20221014162905992](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014162905992.png)

部署流程：

```java
    /**
     * 完成流程的部署操作
     */
    @Test
    public void deploy(){
        Deployment deploy = repositoryService.createDeployment()
                .name("表单01")
                .addClasspathResource("flow/表单-02.bpmn")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
    }
```

启动一个流程实例：普通流程启动

```java
    /**
     * 正常的启动流程
     */
    @Test
    void startFormFlow() throws Exception{
        String processDefId = "Process_1xf3zgp:1:11da0157-4b9a-11ed-8750-c03c59ad2248";
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefId);
    }
```

然后现在流程执行到了`用户任务01`出。我们绑定的有流程表单。我们可以查询看看。先来查询Task对应的表单信息

![image-20221014165547832](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014165547832.png)

```java
    @Test
    void getTaskFormData(){
        String taskId = "bff75f55-4b9a-11ed-9709-c03c59ad2248";
        TaskFormData taskFormData = formService.getTaskFormData(taskId);
        List<FormField> formFields = taskFormData.getFormFields();
        for (FormField formField : formFields) {
            // 获取每个具体的表单域
            System.out.println(formField.getId() + " " + formField.getLabel() + " " + formField.getValue().getValue());
        }
    }
```

![image-20221014163939776](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014163939776.png)

然后可以给表单的相关信息赋值

```java
    @Test
    void setTaskFormData(){
        String taskId = "bff75f55-4b9a-11ed-9709-c03c59ad2248";
        taskService.setVariable(taskId,"days1",5);
        taskService.setVariable(taskId,"reason1","个人原因");
    }
```

然后可以看到表单的信息

![image-20221014165040994](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014165040994.png)



![image-20221014165131403](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014165131403.png)

审批完成到下一个节点

![image-20221014165620888](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014165620888.png)

这时我们再查询下看能不能找到对应的表单信息

![image-20221014165737958](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221014165737958.png)



好了这个是我们讲的直接在流程中内嵌表单的应用



## 2.外置表单

&emsp;&emsp;我们还可以自己定义一个 HTML+JS 的动态表单，然后在审批相关节点的时候渲染展示。

### 2.1 定义一个表单

&emsp;&emsp;我们先定义一个简单的表单

![image-20221015134123018](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221015134123018.png)

### 2.2 流程绑定

&emsp;&emsp;然后在流程设计的时候绑定对应的表单

![image-20221015134202377](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221015134202377.png)

部署流程后启动

![image-20221015134234163](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221015134234163.png)

![image-20221015134256088](D:\desktop\工作目录\01-录课资料\22-camunda\02-课件资料\https://gwzone.oss-cn-beijing.aliyuncs.com/bpmn/image-20221015134256088.png)

在跳转后的地址中有taskId和回调的接口地址