### 什么是领域模型？
领域模型是关于某个特定业务领域的软件模型。通常，领域模型通过对象模型来实现，这些对象同时包含了数据和行为，并且表达了准确的业务含义。

战略设计帮助我们理解哪些投入是最重要的；哪些既有软件资产是可以重新拿来使用的；哪些人应该被加到团队中？
战术设计则帮助我们创建DDD模型中各个部件。

界限上下文 看成是整个应用程序之内的一个概念性边界。这个边界之内的每种领域术语、词组或句子——也即通用语言，都有确定的上下文含义。在边界之外，这些术语可能表示不同的意思。

通用语言 是团队自己创建的公用语言，团队中同时包含领域专家和软件开发人员。 https://www.jianshu.com/p/ec729b949a1c  

#### 判断是否需要使用DDD
![](resources/ddd/ddd_scoring1.jpg)
![](resources/ddd/ddd_scoring2.jpg)

通用语言？

## 定义
#### 实体
#### 值对象
#### 聚合根
#### 领域服务
#### 领域事件




### 领域驱动设计整体架构
- Presentation Layer：表现层，负责显示和接受输入；
- Application Layer(Service)：应用层，很薄的一层，只包含工作流控制逻辑，不包含业务逻辑；
- Domain Layer(Domain)：领域层，包含整个应用的所有业务逻辑；
- Infrastructure Layer：基础层，提供整个应用的基础服务；
![](resources/ddd/architecture.png)

### 定义
- 实体-Entity，DDD中要求实体是唯一的且可持续变化的。意思是说在实体的生命周期内，无论其如何变化，其仍旧是同一个实体。唯一性由唯一的身份标识来决定的。可变性也正反映了实体本身的状态和行为。
- 值对象-Value Objects，度量或描述领域中的一件东西，不可变。
- 领域服务-Domain Services，表示一个无状态的操作，用于实现特定于某个领域的任务，当某个操作不适合放在聚合和值对象上时，最好的方式便是使用领域服务。
- 领域事件-Domain Events，可以理解为是发生在一个特定领域中的事件，希望在同一个领域中其他部分知道并产生后续动作的事件。
- 模块-Modules，指提供特定功能的相对独立的单元。类似 Java 中包。
- 聚合-Aggregate，DDD的核心是聚合。聚合是一种特殊的实体，聚合包装一组高度相关的对象，作为一个数据修改的单元。聚合最外层的对象称为聚合根，它是一个实体。
- 资源库-Repository，表示一个安全的存储区域，通常将聚合实例存放在资源库中，之后通过该资源库获取相同的实例。

### 关于领域（Domain）、领域模型（Domain Model）、边界上下文（Bounded Context）的关系
- 关于领域（Domain）、领域模型（Domain Model）、边界上下文（Bounded Context）的关系
- 领域就是问题域，问题空间；
- 领域模型是一种模型，表达了领域中哪些业务需求以及业务规则必须被满足；
- 每一个领域中的问题，都会有一个对应的领域模型去解决；
- Bounded Context的作用是用来对领域模型进行划分；
- 划分领域就是对问题空间的划分，通俗的理解，就是将大问题拆分为小问题；
- 划分Bounded Context就是将一个大的领域模型划分为多个小的领域模型；
- 可以把Bounded Context看成是一种解决方案空间，所以，Bounded Context也可以理解为是对解决方案空间的划分；
- 理论上，一个Domain可能会对应多个Bounded Context；同样，一个Bounded Context可能也会对应多个Domain；所以他们之间没有绝对的关系。主要是他们划分的依据不同，一个是针对领域（问题空间），一个是针对领域模型（解决方案空间）；理想情况，一个Domain最好对应一个Bounded Context；

### 关于Domain、Sub Domain、Core Domain、Generic Domain，以及Shared Kernal的理解：
- 一个领域（Domain）会拆分为多个子领域（Sub Domain）；
- 子领域中最核心（最重要）的那个叫Core Domain；我们应该讲团队的核心资源用在核心子域上，因为它是产品成败的关键；
- 除了Core Domain外，其他的是支撑子域（Supporting Subdomain）；
- 有些支撑子域比较特殊，因为它解决的是一类通用问题，比如账号和权限；这类子域我们叫做通用子域（Generic Subdomain）；通常，通用子域对应的Bounded Context，会跨域多个子域；
- 多个子领域有时会有相交的部分，我们称作共享内核（Shared Kernel）；体现到代码上，就是同一份代码，在两个领域模型中复用；
- 一般只有Domain比较大的时候，我们才会划分出Sub Domain；

《实战领域驱动设计》
https://www.jianshu.com/c/27b2faf4ed7f  
https://www.cnblogs.com/netfocus/p/4492486.html