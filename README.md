说明
---
#### IDDD由四个子项目组成：

>1.iddd_agilepm 项目是一个以LevelDB这样key-value存储作为持久层的案例，
    LevelDB 是一个纯Java实现: https://github.com/renfeihn/leveldb
    iddd_agilepm并没有使用任何容器(such as Spring).

>2.iddd_collaboration项目是一个使用 Event Sourcing 和
    CQRS的案例.它避免使用了ORM之类框架，展示基于JDBC的查询引擎.
    这样技术虽然有其局限性，但是意味着小而快，无需任何元注解之类配置，虽然不完美。
    iddd_collaboration项目展示它持久Event Sourced写模型和在另外一个线程实现CQRS的读模型.
    使用LevelDB进行事件存储播放而 MySQL用于读模型的存储。也许两者之间有些数据不一致，可实现最终一致性。

>3.The iddd_identityaccess 项目是使用ORM作为持久(Hibernate),
    但是也没有被它干扰，提供一个RESTful客户端接口，通过REST (logs)
    和 RabbitMQ.可发送Domain-Event提醒。

>4.iddd_common 项目提供许多可重用组件，虽然并不试图成为一个框架，但是能提高代码重用


#### 如何编译IDDD_Samples项目：
>1.安装RabbitMQ；安装包在工程根目录tools文件夹
    http://www.cnblogs.com/junrong624/p/4121656.html

>2.启动RabbitMQ；

>3.将IDDD_Samples\iddd_collaboration\src\main\mysql\collaboration.sql导入数据库；

>4.将IDDD_Samples\iddd_common\src\main\mysql\common.sql导入数据库；
  将IDDD_Samples\iddd_common\src\main\mysql\test_common.sql导入数据库；

>5.将IDDD_Samples\iddd_identityaccess\src\main\mysql\iam.sql导入数据库；

>6.修改IDDD_Samples\iddd_collaboration\src\main\resources\applicationContext-collaboration.xml
中数据库相关配置；

>7.修改IDDD_Samples\iddd_common\src\main\resources\applicationContext-common.xml和
      IDDD_Samples\iddd_common\src\main\resources\hibernate.cfg.xml中数据库相关配置；

>8.修改IDDD_Samples\iddd_identityaccess\src\main\resources\hibernate.cfg.xml中数据库相关配置；

#### 小知识点

##### 1.CQRS(命令查询责任分离)架构
        关于CQRS（Command Query Responsibility Segration）架构。
    简单的说，就是一个系统，从架构上把它拆分为两部分：命令处理（写请求）+查询处理（读请求）。
    然后读写两边可以用不同的架构实现，以实现CQ两端（即Command Side，简称C端；Query Side，简称Q端）的分别优化。
    CQRS作为一个读写分离思想的架构，在数据存储方面，没有做过多的约束。所以，我觉得CQRS可以有不同层次的实现，比如：
        1.CQ两端数据库共享，CQ两端只是在上层代码上分离；这种做法，带来的好处是可以让我们的代码读写分离，更好维护，
    且没有CQ两端的数据一致性问题，因为是共享一个数据库的。我个人认为，这种架构很实用，既兼顾了数据的强一致性，又能让代码好维护。
        2.CQ两端数据库和上层代码都分离，然后Q的数据由C端同步过来，一般是通过Domain Event进行同步。同步方式有两种，
    同步或异步，如果需要CQ两端的强一致性，则需要用同步；如果能接受CQ两端数据的最终一致性，则可以使用异步。采用这种方式的架构，
    个人觉得，C端应该采用Event Sourcing（简称ES）模式才有意义，否则就是自己给自己找麻烦。因为这样做你会发现会出现冗余数据，
    同样的数据，在C端的db中有，而在Q端的db中也有。和上面第一种做法相比，我想不到什么好处。
    而采用ES，则所有C端的最新数据全部用Domain Event表达即可；而要查询显示用的数据，则从Q端的ReadDB（关系型数据库）查询即可。

##### 2.依赖倒置原则

        依赖倒置原则（Dependence Inversion Principle，简称DIP）这个名字看着有点别扭，“依赖”还“倒置”，这到底是什么意思？
    依赖倒置原则的原始定义是：
    1.High level modules should not depend upon low level modules.
    2.Both should depend upon abstractions. Abstractions should not depend upon details.
    3.Details should depend upon abstractions。
    翻译过来，包含三层含义：
    1.高层模块不应该依赖低层模块，两者都应该依赖其抽象；
    2.抽象不应该依赖细节；
    3.细节应该依赖抽象。

        高层模块和低层模块容易理解，每一个逻辑的实现都是由原子逻辑组成的，不可分割的原子逻辑就是低层模块，
    原子逻辑的再组装就是高层模块。那什么是抽象，什么又是细节呢？在Java语言中，抽象就是指接口或抽象类，
    两者都是不能直接被实例化的；细节就是实现类，实现接口或继承抽象类而产生的类就是细节，
    其特点就是可以直接被实例化，也就是可以加上一个关键字new产生一个对象。依赖倒置原则在Java语言中的表现就是：
    1.模块间的依赖是通过抽象发生，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或抽象类产生的；
    2.接口或抽象类不依赖于实现类；
    3.实现类依赖接口或抽象类。

    更加精简的定义就是“面向接口编程”——OOD（Object-Oriented Design，面向对象设计）的精髓之一。

##### 3.六边形架构（端口与适配器）

        六边形架构（Hexagonal Architecture），又称为端口和适配器架构风格，其中的“六”具体数字没有特殊的含义，
    仅仅表示一个“量级”的意思，六边形的定义只是方便更加形象的理解。
        我们知道分层架构的重要作用就是避免耦合的出现，经典分层架构和六边形架构都是分层架构的一种，
    但是所发挥的作用会有些不同，经典分层架构更多的精力放在抽象的分离上，每个层的职责分的很明确，
    各个层的依赖关系更加抽象化，从而避免耦合的出现，而在六边形架构中，是用“组件化”的形式来避免耦合的出现，
    每个业务单元尽可能的最小化，然后把这些业务组件集合起来，用一个锤子把他们都拍扁，
    所以，在整个集合中，这些小的业务单元都是“平等的”，这种方式用一个词来概括，那就是“扁平化”。
    由于核心域位于限界上下文中，我们可以使用多种风格的架构。

        那为什么核心域位于限界上下文中，DDD 就可以使用多种风格的架构？我们来分析一下，
    核心域指的是业务系统中的核心业务逻辑，这个通常用通用语言表述，限界上下文是一种边界，
    它包裹的是核心域，也就是说，核心域并不是组件化的形式表现，你可以把它看作是一种聚合概念，
    用限界上下文来进行限定，这个概念在之前的博文中有说明。对于业务系统来说，核心域毫无疑问是核心概念，
    DDD 的专注点也就是它，开发人员和领域专家会花大量的时间去探讨它，但对于架构设计来说，
    核心域只是业务上的专注点，并非是架构设计上的核心点，所以，也可以这样说，DDD 和架构设计，
    其实严格来说应该是两个领域方面的概念，他们的结合才真正构成整体的业务系统，换句话说，
    最烂的架构设计配上最好的 DDD（业务上的），这也是可以的，因为 DDD 专注的是业务实现，而并非是技术实现。

    对于六边形架构，一般会分成三层：
    1.领域层（Domain Layer）：最里面，纯粹的核心业务逻辑，一般不包含任何技术实现或引用。
    2.端口层（Ports Layer）：领域层之外，负责接收与用例相关的所有请求，这些请求负责在领域层中协调工作。
    端口层在端口内部作为领域层的边界，在端口外部则扮演了外部实体的角色。
    3.适配器层（Adapters Layer）：端口层之外，负责以某种格式接收输入、及产生输出。
    比如，对于 HTTP 用户请求，适配器会将转换为对领域层的调用，并将领域层传回的响应进行封送，
    通过 HTTP 传回调用客户端。在适配器层不存在领域逻辑，它的唯一职责就是在外部世界与领域层之间进行技术性的转换。
    适配器能够与端口的某个协议相关联并使用该端口，多个适配器可以使用同一个端口，在切换到某种新的用户界面时，
    可以让新界面与老界面同时使用相同的端口。
        在六边形架构中，领域层和技术没半毛钱关系，可以看作是业务的技术实现，端口层包裹在领域层在外，
    外部要向和领域层“交流”，则必须通过端口层的“首肯”，反过来，领域层向外面“交流”也是一样，但这种方式一般是技术上的。

##### 4.SOA

        SOA（Service-Oriented Architecture）。
        Service 意为服务，面向服务，也就是在 SOA 架构中，服务是核心。业务系统中分布着大量的业务模块，
    这些业务模块进一步提炼就是服务，然后通过规定的服务契约发布出来，这些服务集合起来就是 SOA 的核心，
    服务调用者可以是多样的，Web 端、移动端、桌面端都可以，也就是说它是分布式的架构，这些服务调用者调用的时候，
    要遵守发布者规定的一些契约和协议，而在服务本身或者之间也需要一定的契约和协议用来约束，
    要不然整个服务集合就会乱套，比如服务发布协议要有统一的规范，不能说一个服务是一个规范，
    服务之间也需要进行抽象抽离，尽可能的做到重用性等等。

    SOA 架构原则：
    1.服务封装
    2.服务松耦合（Loosely coupled）- 服务之间的关系最小化，只是互相知道。
    3.服务契约 - 服务按照服务描述文档所定义的服务契约行事。
    4.服务抽象 - 除了服务契约中所描述的内容，服务将对外部隐藏逻辑。
    5.服务的重用性 - 将逻辑分布在不同的服务中，以提高服务的重用性。
    6.服务的可组合性 - 一组服务可以协调工作并组合起来形成一个组合服务。
    7.服务自治 – 服务对所封装的逻辑具有控制权
    8.服务无状态 – 服务将一个活动所需保存的资讯最小化。
    9.服务的可被发现性 – 服务需要对外部提供描述资讯，这样可以通过现有的发现机制发现并访问这些服务。


##### 5.REST

##### 6.
