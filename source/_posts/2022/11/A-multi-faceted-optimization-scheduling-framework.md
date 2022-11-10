---
title: A multi-faceted optimization scheduling framework
date: 2022-11-10 22:00:41
tags:
categories:
---
# 云计算中基于粒子群优化算法的多面优化调度框架

在云计算中，由于各种资源需求和多个作业的执行，在性能和成本优化方面存在重大问题

任务和资源多样性，占用CPU和存储，难以优化

创新：

1.考虑任务对资源需求的差异，提出了资源成本时间轴模型

2.提出了一种基于粒子群优化算法(EPBPSO)的调度模型中的增强型性能预算算法，该算法在实现多面优化调度方面具有很大的优势

研究内容：提出了一种增强的粒子群算法，使其能够根据解的质量进行自我调整。优化性能和成本

贡献：

1.一个模型：为了定义任务对特定资源的需求细节，提出了一个模型，即资源成本时间轴模型(resource Cost Timeline model)。该模型有助于用户预算和资源成本的计算。

2.一个框架：针对性能、成本和可靠性、期限、可扩展性等各个方面，提出了基于粒子群算法的多面调度优化框架。

3.一个算法：在粒子群算法的基础上，提出了一种主要考虑性能和预算成本两个约束条件的增强性能预算的粒子群优化算法。增强性能预算的粒子群算法解决了原粒子群算法容易陷入局部最优的问题。

工作：

首先需要量化任务的成本，以便于计算出每个任务对资源的需求，可以将任务成本和资源成本作为两个参数，由于环境的异质性和资源的多样性，在很多场景中，没有为每个资源提供任务需求，这增加了复杂性。

由于粒子群算法的缺点，没有充分考虑反馈的评价和解的质量，导致了在高维空间的局部最优。本文将代表一个考虑性能和成本两个质量参数的框架，并将其输出与以前的结果相匹配。

# 问题的定义和表述

问题的框架和不同术语的几个定义分别如图1和表1所示

## 任务和资源的定义

首先，认为当前云计算系统中有N个资源R = {R1, R2, R3…RN}，有M个任务T = {T1, T2, T3…TM)。任务被称为cloudlet，云资源被称为虚拟资源。

**定义1**![image-20221110205617740](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110205617740.png)这些参数由用户提交给代理，最后提交给任务管理器。

![image-20221110205635031](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110205635031.png)

**定义2**   ***(Resources)***数据中心中的每个虚拟资源主要由其内存和CPU两个参数定义，![image-20221110210153487](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110210153487.png)这两个参数分别用Memory usage和CPU utilization表示

提出的上述问题的系统框架模型如图1所示。在这里，用户以cloudlet形式给出的请求被提交给Broker。Broker将用户请求提交给任务管理器，任务管理器将用户请求分为从属请求和从属请求

![image-20221110204713335](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110204713335.png)

提出的上述问题的系统框架模型如图1所示。在这里，用户以cloudlet形式给出的请求被提交给Broker。Broker将用户请求提交给任务管理器，任务管理器将用户请求分为从属请求和独立请求，信息最后提交给调度器。

![image-20221110204654048](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110204654048.png)

CPU信息和内存负载信息由本地资源管理器收集，最后将这些信息提交给全局资源管理器。本地虚拟机的监视和管理由本地资源管理器完成(例如，特定任务在特定资源上花费的时间)。最后，从资源成本时间轴模型中，利用全局资源管理器提供的先验信息来评估成本。

## 资源成本时间表模型

首先，建立了**用户预算**与**资源成本**的关系模型;提出了一个基于**性能**和**预算约束**的多层面优化调度框架。

### 资源成本模型

由于云计算中任务和资源的多样性，反映任务成本和用户预算成本与资源成本之间的关系是有效的。我们会发现不同资源的成本是不同的，任务的成本也是不同的，因为有些任务需要很多CPU资源，而有些任务则依赖于内存需求。

在这个模型下，考虑三种类型的成本:

a) Memory Cost  
b) CPU Cost  
c) Total Budget Cost

**Memory Cost** 
内存开销包括内存的最小开销，即1gb，任务在特定资源tij上运行的持续时间，与内存传输相关的开销，Mj是特定资源j的内存开销。

![image-20221110213255728](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110213255728.png)

**CPU Cost**
类似地，CPU成本也将包括最小利用成本Cmin、任务持续时间tij、与CPU传输相关的成本和CPU成本Cj。

![image-20221110213702220](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110213702220.png)

![image-20221110213710739](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110213710739.png)

**Total Budget Cost** 
总预算成本是CPU总成本和内存总成本的总和。

![image-20221110213745530](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110213745530.png)

因此，特定资源的最终预算在等式中计算。(6)借助于等式计算的总预算成本(5).

![image-20221110213836899](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110213836899.png)

### 时间轴模型

在时间线模型中，我们考虑任务在多个资源上的总执行时间，并且总执行时间不应该超过根据服务水平协议(SLA)的截止时间，一个特定任务使用的资源可以达到n。

![image-20221110214635032](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110214635032.png)

### 调度模型中基于粒子群优化的多方位优化调度框架

优化调度问题的几个参数是性能、成本、期限和预算成本。寻找这类组合问题的最优解是困难的。粒子群优化算法在寻找这种组合问题的解方面具有优势。粒子群算法已被用于各种各种调度问题，与其他基于自然的算法相比，取得了较好的结果。然而，粒子群算法有陷入局部最优解的缺点。因此，本文提出了一种增强型粒子群优化算法。使用合适的函数，它提供解决方案的质量和对评估结果的反馈。因此，它只是避开了局部最优。

![image-20221110215653038](https://gwzone.oss-cn-beijing.aliyuncs.com/typora-user-images/image-20221110215653038.png)