
1.  Aggregate Framework概述

    Aggregate Framework是基于DDD和CQRS思想而开发的一个领域驱动框架。其主要目标是方便开发人员运用DDD和CQRS思想来构建复杂的、可扩展的应用系统。该框架提供了最核心的构建块的实现，比如Aggregate、Repository和Event。此外，该框架支持与Spring集成，提供使用annotation的方式让开发人员方便地注册事件及定义事件处理，使用Spring事务管理器管理事务时，支持Unit Of Work模式存储数据。

2.  核心概念

2.1 Aggregate

    Aggregate(聚合)定义了一组具有内聚关系的相关对象的集合，是一致性修改数据的单元。聚合里包含有聚合根，实体、值对象。在Aggregate Framework中，聚合最核心的接口是DomainObject，聚合对象都实现该接口。另一个接口是AggregteRoot，该接口继承自DomainObject, 聚合根实现该接口。AbstractDomainObject抽象类实现了DomainObject接口, 并提供了对部分方法的重载。AbstractAggregateRoot继承AbstractDomainObject，在AbstractDomainObject上新增了对Event的支持。为方便定义聚合对象，AbstractSimpleAggregateRoot和AbstractSimpleDomainObject分别对AbstractAggregateRoot和AbstractDomainObject进行了简单实现，提供对接口的默认实现，让开发人员不需关系内部事件注册及发布机制。

2.2 Repository

    在Repository构建块中，最核心的接口莫过于Repository。它是个标记接口，CrudRepository继承该接口，并为聚合对象提供了专门的CRUD方法。AggreateRepository继承CrudRepository, 限制操作的实体是聚合根。AbstractAggregateRepository提供了对AggregateReposiotry的部分实现，将事件的发布进行了简单处理。TraversalAggregateRepository继承了AbstractAggregateRepository，同时实现了对聚合里对象的成员变量进行遍历并调用对应的DAO方法进行CRUD操作。DaoAwareAggregateRepository继承自TraversalAggregateRepository，实现了基于Spring获取DAO依赖实现聚合对象的遍历CRUD操作。

2.3 Event

    Aggregate Framework里有两种事件，一个是Domain Event,另一个是Application Event。Domain Event表达业务上的事件，而Application Event是系统层面的事件。当有事务支持时，Domain Event在事务边界内发布，而Applicatin Event在事务结束后发布。AggregateRoot接口里定义了两个方法，一个是applyDomainEvent，用来注册Domain Event, 另一个时applyApplicationEvent,用来注册Application Event。

3.  使用Aggregate Framework开发示例

     示例通过对一个聚合的操作及事件处理来阐述，代码可以参考aggregate-framework-test项目。该聚合模型里 Order是聚合根，SeatAvailability和Payment为聚合里的实体，Order与SeatAvailability是一对多关系，与Payment是1对1关系，为了阐述问题，SeatAvailabilit引用了Payment以示例聚合内对象相互引用。

3.1 定义领域模型

    Order继承AbstractSimpleAggregateRoot, 其聚合了seatAvailabilities和payment。在seatAvailabilities上添加DaoAwareQuery annotation表示当需要根据Order的主键获取对应的SeatAvailability实体时需要调用SeatAvailabiltiy对应DAO的哪个方法。SeatAvailablity聚合了payment，在一个聚合中，SeatAvailability引用的payment与Order引用的payment可以是同一个。

3.2 定义Repository

    OrderRepository继承DaoAwareAggregateRepository，并定义操作的AggregateRoot实体类型为Order。

3.3 定义聚合对象的DAO

    每个聚合对象都有一个对应的DAO。聚合根的DAO为OrderDao,继承AggregateRootDao, SeatAvilability和Payment的Dao继承DomainObjectDao。

3.4 定义事件及处理

    示例中定义了三个事件：OrderCreatedEvent,OrderUpdatedEvent,OrderUpdatedApplicationEvent。OrderCreatedEvent和OrderUpdatedEvent被applyDomainEvent注册，表示领域事件，OrderCreatedEvent在Order初始化时注册，OrderUpdatedEvent则在updateContent里注册。updateContent里还使用applyApplicationEvent注册了OrderUpdatedApplicationEvent，表注册的事件为系统事件。使用Spring管理并将aggregate-framework-context.xml载入Spring后，标识了EventHandler annotation的方法将自动为事件处理方法，其对应的事件与方法入参类型对应。示例中OrderHandler为事件处理类，handleOrderCreatedEvent、handleOrderUpdatedEvent、handleOrderUpdatedApplicationEvent分别处理OrderCreatedEvent、OrderUpdatedEvent和OrderUpdatedApplicationEvent事件。

3.5 事务支持

    在调用Repository方法时，可以选择事务以实现聚合的原子化操作。使用Spring管理事务时，需配置SessionDataSourceTransactionManager为事务管理器。

3.6 Test

    最后在aggregate-framework-test项目里，OrderRepositoryTest里有各种unit test。




