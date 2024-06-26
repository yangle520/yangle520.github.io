---
layout: post
title:  软件复杂度
date:   2024-02-26 00:00:00 +0800
categories: 系统设计
tag: 系统设计
author: YangLe
---

## 复杂性

复杂性的两个度量

1. 系统是不是难以理解
2. 系统是不是难以修改

模糊性与依赖性是引起复杂性的2个主要因素



### 复杂性的表现形式

#### 变更放大

1. 指得是看似简单的变更需要在许多不同地方进行代码修改
2. 缺少内聚与收拢
3. 当需要对某段业务进行调整时，需要改动多个模块以适应业务的发展



#### 认知负荷

1. 是指开发人员需要多少知识才能完成一项任务
2. 使用功能性框架时，我们希望它操作简单，部署复杂系统时，我们希望它架构清晰，其实都是降低一项任务所需的成本。
3. 盲目的追求高端技术，设计复杂系统，增加学习与理解成本都属于本末倒置的一种



#### 未知的未知

1. 是指必须修改哪些代码才能完成任务，或者说开发人员必须获得哪些信息才能成功地执行任务
2. 你不知道改动的这行代码是否能让程序正常运转，也不知道这行代码的改动是否又会引发新的问题



## 软件架构治理复杂度

分布式架构、SOA、微服务、FaaS、Service Mesh 等等。所有的软件架构万变不离其宗，都在致力解决软件的复杂性



### 架构的本质

编程范式指的是程序的编写模式，软件架构发展到今天只出现过3 种编程范式，分别是**结构化编程**，**面向对象编程**与**函数式编程**

- 结构化编程取消 goto 移除跳转语句，对程序控制权的直接转移进行了限制和规范
- 面向对象编程限制指针的使用，对程序控制权的间接转移进行了限制和规范
- 函数式编程以 λ 演算法为核心思想，对程序中的赋值进行了限制和规范

> 面向对象的五大设计原则：
>
> 1. 单一职责原则(Single Responsibilities Principle)： 一个类，最好只做一件事，只有一个引起它的变化
> 2. 开放封闭原则（Open Closed Principle）： 软件实体应该是可扩展的，而不可修改的。也就是，对扩展开放，对修改封闭的
> 3. 里氏替换原则（Liskov Substituion Principle）：子类必须能够替换其基类
> 4. 依赖倒置原则（Dependecy-Inversion Principle）：高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象
> 5. 接口隔离原则（Interface-Segregation Principle）：使用多个小的专门的接口，而不要使用一个大的总接口。

**软件的本质是约束**

软件发展没有告诉我们应该怎么做，而是教会我们不该做什么



> A condition that is often incorrectly labeled software maintenance. To be more precise, it is maintenance when we correct errors; it is evolution when we respond to changing requirements.
>
> 在软件发展过程里，只有我们修正错误时，才是维护；在我们应对改变的需求时，这是演进。
>
> Grady Booch 《Object-Oriented Analysis and Design with Applications》



> 当我们使用一些极端的手段来保持古老而陈腐的软件继续工作时，这确实是一种苟且。我们小心翼翼、集成测试、灰度发布、及时回滚等等，我们没有在“维护”他们，而是以一种丑陋的方式让这些丑陋的代码继续能够成功苟且下去。
>
> Grady Booch 《Object-Oriented Analysis and Design with Applications》



> Comments do not make up for bad code
>
> 注释不是对劣质代码的补救
>
> Martin Fowler 《Clean Code》



### 注释

1. 无法精准命名
   命名的含义是抽象实体隐藏细节，我们不能在一个名字上赋予它全部的信息，而必要的注释可以完美的进行辅佐
2. 设计思想的阐述
   代码只能实现设计不能阐述设计，这也是为什么一些复杂的架构设计我们需要文档的支撑而非代码的‘自解释’，在文档与代码之间的空隙，由注释来填补
3. 母语的力量
   这点尤其适合我们中国人，有时并不是因为注释少代码多，所以我们下意识会首先看代码。而是我们几十年感受的文化，让我们对中文与ABC具有完全不一样的感观

### 追求优雅

> The goal of software architecture is to minimize the human resources requiredto build and maintain the required system.
>
> 软件架构的终极目标是，用最小的人力成本来满足构建和维护该系统的需求。
>
> Robert C.Martin 《Clean Architecture》



### 设计两次

这里“设计两次”的意思是无论设计一个类，模块还是功能，在设计的时候仔细思考，除了当前的方案，还有那些其它的选择。在众多设计中比较，列出各自的优缺点，然后选出最佳方案。就是对于设计方案，都有两个或者两个以上的选择。

对于大牛而言，也许设计方案显而易见，于是觉得没有必要在不同方案中做遴选。然而这并不是一个好的习惯，这说明，你没有在处理更困难的问题，问题对于你而言太简单了。这不是一个好的现象，因为上坡路总是很难走。当你面对困难的问题的时候，通过对不同设计方案的学习和思考，你会成长到更高的一个层次。

我的解读：在管理理论上有一个叫**彼得原理**，就是“在一个等级制度中，每个人趋向于上升到他所不能胜任的地位”。程序员也面临同样的问题，当你的经验和资历不断的提高，你总会遇到你所不能胜任的问题，这个时候就需要通过不断的学习，提高自己。当然也有可能所处的环境无法给你更具挑战的问题。这个时候你就需要考虑，你的下一站在哪里？







