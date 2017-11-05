说明
---
#### IDDD由四个子项目组成：

>1.iddd_agilepm 项目是一个以LevelDB这样key-value存储作为持久层的案例，
    LevelDB 是一个纯Java实现: https://github.com/dain/leveldb
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
>1.安装RabbitMQ；
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

##### 1.CQRS架构
        关于CQRS（Command Query Responsibility Segration）架构。
    简单的说，就是一个系统，从架构上把它拆分为两部分：命令处理（写请求）+查询处理（读请求）。
    然后读写两边可以用不同的架构实现，以实现CQ两端（即Command Side，简称C端；Query Side，简称Q端）的分别优化。
    CQRS作为一个读写分离思想的架构，在数据存储方面，没有做过多的约束。所以，我觉得CQRS可以有不同层次的实现，比如：
        (1)CQ两端数据库共享，CQ两端只是在上层代码上分离；这种做法，带来的好处是可以让我们的代码读写分离，更好维护，
    且没有CQ两端的数据一致性问题，因为是共享一个数据库的。我个人认为，这种架构很实用，既兼顾了数据的强一致性，又能让代码好维护。
        (2)CQ两端数据库和上层代码都分离，然后Q的数据由C端同步过来，一般是通过Domain Event进行同步。同步方式有两种，
    同步或异步，如果需要CQ两端的强一致性，则需要用同步；如果能接受CQ两端数据的最终一致性，则可以使用异步。采用这种方式的架构，
    个人觉得，C端应该采用Event Sourcing（简称ES）模式才有意义，否则就是自己给自己找麻烦。因为这样做你会发现会出现冗余数据，
    同样的数据，在C端的db中有，而在Q端的db中也有。和上面第一种做法相比，我想不到什么好处。
    而采用ES，则所有C端的最新数据全部用Domain Event表达即可；而要查询显示用的数据，则从Q端的ReadDB（关系型数据库）查询即可。



