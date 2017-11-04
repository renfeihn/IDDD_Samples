These are the sample Bounded Contexts from the book
"Implementing Domain-Driven Design" by Vaughn Vernon:

http://vaughnvernon.co/?page_id=168

The models and surrounding architectural mechanisms
may be in various states of flux as the are refined
over time. Some tests may be incomplete. The code is
not meant to be a reflection of a production quality
work, but rather as a set of reference projects for
the book.

Points of Interest
==================

The iddd_agilepm project uses a key-value store as
its underlying persistence mechanism, and in particular
is LevelDB. Actually the LevelDB in use is a pure Java
implementation: https://github.com/dain/leveldb

Currently iddd_agilepm doesn't employ a container of
any kind (such as Spring).

The iddd_collaboration project uses Event Sourcing and
CQRS. It purposely avoids the use of an object-relational
mapper, showing that a simple JDBC-based query engine
and DTO matter can be used instead. This technique does
have its limitations, but it is meant to be small and fast
and require no configuration or annotations. It is not
meant to be perfect.

It may be helpful to make one additional mental note on
the iddd_collaboration CQRS implementation. To keep the
example simple it persists the Event Sourced write model
and the CQRS read model in one thread. Since two different
stores are used--LevelDB for the Event Journal and MySQL
for the read model--there may be very slight opportunities
for inconsistency, but not much. The idea was to keep the
two models as close to consistent as possible without
using the same data storage (and transaction) for both.
Two different storage mechanisms were used purposely to
demonstrate that they can be separate.

The iddd_identityaccess project uses object-relational
mapping (Hibernate), but so as not to leave it "boring" it
provides a RESTful client interface and even publishes
Domain-Event notifications via REST (logs) and RabbitMQ.

Finally the iddd_common project provides a number of reusable
components. This is not an attempt to be a framework, but
just leverages reuse to the degree that code copying doesn't
liter each project. This is not a recommendation, but it
did work well and save a considerable amount of work while
producing the samples.

Usage
=====

Requires
--------

- Java 7 (8+ does not work)
- MySQL Client + Server
- RabbitMQ

Setup (with Docker)
-------------------

To make it easy to run the tests and it requirements,
the `startContainers.sh` script is provided. Which
will start a:
- MySQL Server container
- RabbitMQ Server container
- RabbitMQ Management container

If the `mysql` command is available, which is the mysql client,
also the required SQL scripts will be imported into the MySQL
Server.

If you use the `startContainers.sh` script, you don't need
MySQL Server and RabbitMQ installed locally. Instead,
Docker needs to be installed as the script will start
MySQL and RabbitMQ in Docker containers.

Build
------

You can build the project by running:

```
./gradlew build
```

This automatically downloads Gradle and builds the project, including running the tests.

The Gradle build using Maven repositories was provided by
Michael Andrews (Github michaelajr and Twitter @MichaelAJr).
Thanks much!


I hope you benefit from the samples.

Vaughn Vernon
Author: Implementing Domain-Driven Design
Twitter: @VaughnVernon
http://vaughnvernon.co/


中文说明
------------------
IDDD由四个子项目组成：

1.iddd_agilepm 项目是一个以LevelDB这样key-value存储作为持久层的案例，
    LevelDB 是一个纯Java实现: https://github.com/dain/leveldb
    iddd_agilepm并没有使用任何容器(such as Spring).

2.iddd_collaboration项目是一个使用 Event Sourcing 和
    CQRS的案例.它避免使用了ORM之类框架，展示基于JDBC的查询引擎.
    这样技术虽然有其局限性，但是意味着小而快，无需任何元注解之类配置，虽然不完美。
    iddd_collaboration项目展示它持久Event Sourced写模型和在另外一个线程实现CQRS的读模型.
    使用LevelDB进行事件存储播放而 MySQL用于读模型的存储。也许两者之间有些数据不一致，可实现最终一致性。

3.The iddd_identityaccess 项目是使用ORM作为持久(Hibernate),
    但是也没有被它干扰，提供一个RESTful客户端接口，通过REST (logs)
    和 RabbitMQ.可发送Domain-Event提醒。

4.iddd_common 项目提供许多可重用组件，虽然并不试图成为一个框架，但是能提高代码重用


如何编译IDDD_Samples_by_laungcisin项目：
1.安装RabbitMQ；
    http://www.cnblogs.com/junrong624/p/4121656.html
2.启动RabbitMQ；
3.将IDDD_Samples_by_laungcisin\iddd_collaboration\src\main\mysql\collaboration.sql导入数据库；
4.将IDDD_Samples_by_laungcisin\iddd_common\src\main\mysql\common.sql导入数据库；
  将IDDD_Samples_by_laungcisin\iddd_common\src\main\mysql\test_common.sql导入数据库；
5.将IDDD_Samples_by_laungcisin\iddd_identityaccess\src\main\mysql\iam.sql导入数据库；

6.修改IDDD_Samples_by_laungcisin\iddd_collaboration\src\main\resources\applicationContext-collaboration.xml
中数据库相关配置；

7.修改IDDD_Samples_by_laungcisin\iddd_common\src\main\resources\applicationContext-common.xml和
      IDDD_Samples_by_laungcisin\iddd_common\src\main\resources\hibernate.cfg.xml中数据库相关配置；

8.修改IDDD_Samples_by_laungcisin\iddd_identityaccess\src\main\resources\hibernate.cfg.xml中数据库相关配置；

