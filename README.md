# 知识点
### 1.项目的技术应用

### 2.mysql索引应该如何设计
索引是对数据库表中一个或者多个列的值进行排序的结构，所以索引将占用磁盘空间，并且影响数据的更新速度，因为每一次更新数据都要重构索引。
- 经常用到的查询字段加索引，多个查询条件经常出现的时候，可用考虑建联合索引，并把蛇别率最高的字段放在最左，因为mysql遵循最左优先，联合索引从最左边开始匹配
- 索引要适合，不要设置过多的索引
- 频繁进行写入更新操作的表，不要建立太多索引
- 数据量比较大的字段不要设置索引，会占用大量存储空间，影响数据更新插入操作效率
- 索引字段，尽量避免NULL，应该指定列为NOT NULL。在MySQL中含有空值的列很难进行查询优化。因为它们使得索引、索引的统计信息以及比较运算更加复杂

### 3.线程池核心参数，线程池的状态
##### 核心参数
- corePoolSize核心线程数量
-  maximumPoolSize 最大允许线程数量
- keepAliveTime、unit 超出线程的存活时间
- workQueue 任务队列
- threadFactory 线程工厂，用于创建线程
- handler 任务拒绝策略

##### 状态
- RUNNING：能接受新提交的任务，并且也能处理阻塞队列中的任务。调用shutdown()方法进入SHUTDOWN状态。调用shutdownNow()方法进入STOP状态。
- SHUTDOWN：关闭状态，不再接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务。当阻塞队列为空，线程池中的工作线程数量为0，转入TIDYING状态。
- STOP：不能接受新任务，也不处理队列中的任务，会中断正在处理的线程。当线程池中工作线程数据为0转为TIDYING状态
- TIDYING：所有的任务都已终止了，workerCount（有效线程数)为0。
调用teminated()方法转为TERMINATED状态。
- TERMINATED：在teminated()方法执行玩之后进入该状态

### 4.mybatis原理
mybatis是一款基于java的持久层框架，支持自定义SQL查询、存储过程和高级映射。mybatis其实就是对jdbc的封装和扩展，jdbc实现查询有七个步骤：
- 1.加载JDBC驱动
- 2.建立并获取数据库连接
- 3.创建 JDBC Statements 对象
- 4.设置SQL语句的传入参数
- 5.执行SQL语句并获得查询结果
- 6.对查询结果进行转换处理并将处理结果返回
- 7.释放相关资源（关闭Connection，关闭Statement，关闭ResultSet

mybatis的查询步骤：
- 1.创建数据库连接池基本信息，并尝试连接，连接成功，进入下一步。
- 2.将数据库连接池的基本信息传给SQLSessionFactory，进行Configuation的配置，获取数据库连接
- 3.传入我们自己设置的*.mappers.xml这些Bean对象的信息，Mybatis获取信息后创建SQLSessionFactory后返回SQLSession给我们
- 4.SQLSession里面是我们的mxl里面的SQL对象信息。当我们通过注解或者JPA的方式调用。则会触发SQLSession进行SQL的解析。也即是调用了ParamHandler进行参数映射。
- 5.SQL解析后Mybatis会根据内部规则重新动态地生成自己的SQL
- 6.执行SQL，通过Executor进行原生JDBC数据接口的调用。Executor里面封装了原生数据库的基本方法接口的ID
- 7.拿到结果后通过resultHandler和typeHandler进行参数映射和配置，返回给代理层


### 5.rabbitMq集群原理
- 普通集群模式<br>
集群中各个节点之间只会相互同步元数据,消息数据不会被同步。内部可以实现转发,机器挂掉，无法高可用。
- 镜像队列模式
节点之间不仅仅会同步元数据，消息内容也会在镜像节点间同步，可用性更高。同步数据之间也会带来网络开销从而在一定程度上会影响到性能。

### 6.rabbitMq延迟队列
利用死信队列实现延迟消息
定义两个队列A和B，和交换机（exchange）X，定义A的“x-dead-letter-exchange”为X。然后建立B与X的bind关系（路由规则需要自己把握），我们发送的消息到达A，然后发送的时候定义消息的存活时间（这个时间就是期望多久收到消息的时间），之后我们并不从A消费消息直到消息过期。然后消息就会被投递到交换机X，自然就能进入队列B。所以我们从B消费就能实现延迟消息的效果啦。

### 7.分布式锁如何设计？有哪些应用场景？
分布式CAP理论：任何一个分布式系统都无法同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance），最多只能同时满足两项。
在很多分布式场景下，要求保证数据的一致性，就可以使用分布式锁。
##### 分布式锁的要求：
- 可以保证在分布式部署的应用集群中，同一个方法在同一时间只能被一台机器上的一个线程执行。
- 这把锁要是一把可重入锁（避免死锁）
- 这把锁最好是一把阻塞锁（根据业务需求考虑要不要这条）
- 这把锁最好是一把公平锁（根据业务需求考虑要不要这条）
- 有高可用的获取锁和释放锁功能
- 获取锁和释放锁的性能要好

##### 分布式锁方案：
- 基于 数据库 做分布式锁
- 基于 Redis 做分布式锁
- 基于 ZooKeeper 做分布式锁

##### 对比
- 数据库分布式锁实现
缺点：1.db操作性能较差，并且有锁表的风险
2.非阻塞操作失败后，需要轮询，占用cpu资源;
3.长时间不commit或者长时间轮询，可能会占用较多连接资源

- Redis(缓存)分布式锁实现
缺点：1.锁删除失败 过期时间不好控制
2.非阻塞，操作失败后，需要轮询，占用cpu资源;

- ZK分布式锁实现
缺点：性能不如redis实现，主要原因是写操作（获取锁释放锁）都需要在Leader上执行，然后同步到follower。

总之：ZooKeeper有较好的性能和可靠性。

从理解的难易程度角度（从低到高）数据库 > 缓存 > Zookeeper
从实现的复杂性角度（从低到高）Zookeeper >= 缓存 > 数据库
从性能角度（从高到低）缓存 > Zookeeper >= 数据库
从可靠性角度（从高到低）Zookeeper > 缓存 > 数据库

场景：秒杀，促销

#### 8.springMVC
springMVC是一款轻量级的web框架。使用了mvc架构模式的思想，将web层进行职责解耦。
springMVC分为三层：模型层(Model)、视图层(View)、控件器(Controller)
视图层即前端界面，模型层则是指实体类，service,dao，控制器即是controller。
spring处理请求流：
- 第一步：用户发送请求到前端控制器（DispatcherServlet）。

- 第二步：前端控制器请求 HandlerMapping 查找 Handler，可以根据 xml 配置、注解进行查找。

- 第三步： 处理器映射器 HandlerMapping 向前端控制器返回 Handler

- 第四步：前端控制器调用处理器适配器去执行 Handler

- 第五步：处理器适配器执行 Handler

- 第六步：Handler 执行完成后给适配器返回 ModelAndView

- 第七步：处理器适配器向前端控制器返回 ModelAndView。ModelAndView是SpringMVC框架的一个底层对象，包括 Model 和 View

- 第八步：前端控制器请求视图解析器去进行视图解析，根据逻辑视图名来解析真正的视图。

- 第九步：视图解析器向前端控制器返回 view

- 第十步：前端控制器进行视图渲染，就是将模型数据（在 ModelAndView 对象中）填充到 request 域

- 第十一步：前端控制器向用户响应结果

### 9.redis原理
Redis本质上是一个Key-Value类型的内存数据库,整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据flush到硬盘上进行保存。



### 10.rocketMQ怎么实现分布式事务的，下游失败了怎么解决，rocketMQ怎么保证高可用？
场景：转账业务，一个是操作用户A扣钱，一个是操作用户B加钱，用户A的数据在集群A中，用户B在集群B。

RocketMQ事务方案
RocketMq消息中间件把消息分为两个阶段：预备阶段和确认阶段
预报阶段：主要发一个消息到rocketmq，但该消息只储存在commitlog中，但consumeQueue中不可见，也就是消费端（订阅端）无法看到此消息。
确认阶段：把prepared消息保存到consumeQueue中，即让消费端可以看到此消息，也就是可以消费此消息。

流程：
1、在扣款之前，先发送预备消息
2、发送预备消息成功后，执行本地扣款事务
3、扣款成功后，再发送确认消息
4、消息端（加钱业务）可以看到确认消息，消费此消息，进行加钱

异常场景：
异常1：如果发送预备消息失败，下面的流程不会走下去；这个是正常的
异常2：如果发送预备消息成功，但执行本地事务失败；这个也没有问题，因为此预备消息不会被消费端订阅到，消费端不会执行业务。
异常3：如果发送预备消息成功，执行本地事务成功，但发送确认消息失败；这个就有问题了，因为用户A扣款成功了，但加钱业务没有订阅到确认消息，无法加钱。这里出现了数据不一致。

RocketMQ回查：
RocketMq会定时遍历commitlog中的预备消息。因为预备消息最终肯定会变为commit消息或Rollback消息，所以遍历预备消息去回查本地业务的执行状态，如果发现本地业务没有执行成功就rollBack，如果执行成功就发送commit消息。上面的异常3，发送预备消息成功，本地扣款事务成功，但发送确认消息失败；因为RocketMq会进行回查预备消息，在回查后发现业务已经扣款成功了，就补发“发送commit确认消息”；这样加钱业务就可以订阅此消息了。

回查判断业务是否成功：
计一张Transaction表，将业务表和Transaction绑定在同一个本地事务中，如果扣款本地事务成功时，Transaction中应当已经记录该TransactionId的状态为「已完成」。当RocketMq回查时，只需要检查对应的TransactionId的状态是否是「已完成」就好，而不用关心具体的业务数据。

### 11.分库分表的操作，按照什么分
水平分库，按照字段拆分
水平分表，按照字段拆分

### HashMap,ConcurrentHashap,ArrayList和CopyOnWriteList原理，如何加锁
- HashMap<br>
JDK1.8 之前 HashMap 底层是 数组和链表
JDK1.8 HashMap 底层是 数组和链表和红黑树。
加锁用synchronized，Lock
- ConcurrentHashMap<br>
1.7
数据结构：数组＋单链表
并发机制：采用分段锁机制细化锁粒度，降低阻塞，从而提高并发性。

1.8
数据结构：数组＋单链表＋红黑树
并发机制：取消分段锁，之后基于 CAS + synchronized 实现。
- ArrayList <br>
ArrayList 的底层是数组队列，相当于动态数组，不保证线程安全.
加锁：Collections.synchronizedList
- CopyOnWriteArrayList <br>
内部维护了一个数组，成员变量array就指向这个内部数组，所有的读操作都是基于 array 进行的
执行写时复制操作，需要使用可重入锁加锁,reentrantLock
读操作是不用加锁的，性能很高

### 13.redis集群架构有哪些，原理是啥
- Replication+Sentinel<br>
这里Sentinel的作用有三个:
监控:Sentinel 会不断的检查主服务器和从服务器是否正常运行。
通知:当被监控的某个redis服务器出现问题，Sentinel通过API脚本向管理员或者其他的应用程序发送通知。

自动故障转移:当主节点不能正常工作时，Sentinel会开始一次自动的故障转移操作，它会将与失效主节点是主从关系 的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点。
工作原理就是，当Master宕机的时候，Sentinel会选举出新的Master，并根据Sentinel中client-reconfig-script脚本配置的内容，去动态修改VIP(虚拟IP)，将VIP(虚拟IP)指向新的Master。我们的客户端就连向指定的VIP即可！
缺陷:
(1)主从切换的过程中会丢数据
(2)Redis只能单点写，不能水平扩容
- Proxy+Replication+Sentinel<br>
工作原理如下
前端使用Twemproxy+KeepAlived做代理，将其后端的多台Redis实例分片进行统一管理与分配
每一个分片节点的Slave都是Master的副本且只读
Sentinel持续不断的监控每个分片节点的Master，当Master出现故障且不可用状态时，Sentinel会通知/启动自动故障转移等动作
Sentinel 可以在发生故障转移动作后触发相应脚本（通过 client-reconfig-script 参数配置 ），脚本获取到最新的Master来修改Twemproxy配置
缺陷:
(1)部署结构超级复杂
(2)可扩展性差，进行扩缩容需要手动干预
(3)运维不方便

- Redis Cluster
工作原理如下

客户端与Redis节点直连,不需要中间Proxy层，直接连接任意一个Master节点
根据公式HASH_SLOT=CRC16(key) mod 16384，计算出映射到哪个分片上，然后Redis会去相应的节点进行操作
具有如下优点:
(1)无需Sentinel哨兵监控，如果Master挂了，Redis Cluster内部自动将Slave切换Master
(2)可以进行水平扩容
(3)支持自动化迁移，当出现某个Slave宕机了，那么就只有Master了，这时候的高可用性就无法很好的保证了，万一master也宕机了，咋办呢？ 针对这种情况，如果说其他Master有多余的Slave ，集群自动把多余的Slave迁移到没有Slave的Master 中。

缺点:
(1)批量操作是个坑
(2)资源隔离性较差，容易出现相互影响的情况。

### 14.redis分布式锁原理，有什么缺点
- 基于 redis 的 setnx()、expire() 方法做分布式锁。会出现死锁
- 基于 redis 的 setnx()、get()、getset()方法做分布式锁
- 基于 Redlock 做分布式锁
优点：性能高
缺点：
失效时间设置多长时间为好？如何设置的失效时间太短，方法没等执行完，锁就自动释放了，那么就会产生并发问题。如果设置的时间太长，其他获取锁的线程就可能要平白的多等一段时间
- 基于 redisson 做分布式锁


### 15.dubbo的注册原理，dubbo的调用流程
##### 基于ZooKeeper解析Dubbo注册中心实现原理
  ①当服务端启动完成后，会在ZooKeeper上注册到/dubbo这个节点下；
   ②消费端，在xml中配置<dubbo:reference>远程服务调用，如果改地址发生变化，都会去通过watch监听/dubbo节点下的该接口名的节点变化；
    ③消费端访问服务端时，会拿到该节点下的所有子节点，当子节点发生变化时，Watch监听会通知给消费端。消费端此时拿到服务端注册在ZooKeeper中的节点地址信息列表，然后根据地址列表对应的服务去发布。如果服务端采用的是集群，那么消费端在得到该服务的过程中，就会通过负载均衡的算法来完成具体的调用过程，Dubbo会对这个地址做负载均衡算法，最后选择一个可用的地址，然后发起调用

### 16.限流的具体实现
- 固定窗口计数器算法<br>
给定一个变量counter来记录处理的请求数量，当1分钟之内处理一个请求之后counter+1，1分钟之内的如果counter=100的话，后续的请求就会被全部拒绝。等到 1分钟结束后，将counter回归成0，重新开始计数（ps：只要过了一个周期就讲counter回归成0）。

这种限流算法无法保证限流速率，因而无法保证突然激增的流量。比如我们限制一个接口一分钟只能访问10次的话，前半分钟一个请求没有接收，后半分钟接收了10个请求。

-  滑动窗口计数器算法<br>
求，我们可以把 1 分钟分为60个窗口。每隔1秒移动一次，每个窗口一秒只能处理 不大于 60(请求数)/60（窗口数） 的请求， 如果当前窗口的请求计数总和超过了限制的数量的话就不再处理其他请求。

很显然：当滑动窗口的格子划分的越多，滑动窗口的滚动就越平滑，限流的统计就会越精确。
-  漏桶算法<br>
我们可以把发请求的动作比作成注水到桶中，我们处理请求的过程可以比喻为 漏桶漏水 。我们往桶中以任意速率流入水，以一定速率流出水。当水超过桶流量则丢弃，因为桶容量是不变的，保证了整体的速率。

如果想要实现这个算法的话也很简单，准备一个队列用来保存请求，然后我们定期从队列中拿请求来执行就好了。
-  令牌桶算法<br>
令牌桶算法也比较简单。和漏桶算法算法一样，我们的主角还是桶（这限流算法和桶过不去啊）。不过现在桶里装的是令牌了，请求在被处理之前需要拿到一个令牌，请求处理完毕之后将这个令牌丢弃（删除）。

我们根据限流大小，按照一定的速率往桶里添加令牌即可！

### 18.锁的升级过程
锁状态一种有四种，从级别由低到高依次是：无锁、偏向锁，轻量级锁，重量级锁，锁状态只能升级，不能降级
无锁->偏向锁：当有一个线程访问同步块时升级成偏向锁
偏向锁->轻量级锁：有锁竞争时升级为轻量级锁
轻量级锁->重量级锁：自旋十次失败升级为重量级锁

### 19.mysql用索引的时候查询过程是怎么样的？如果a,b,c三个索引，都是独立索引，用到哪个？

有主键参与查询，直接查询主键索引，没有主键索引，有辅助索引的时候先根据辅助索引找到对应的主键索引，再根据主键索引查找具体的数据行。三个独立的索引进行查询，会看哪个索引的区分度大，筛选更快就使用哪个，如果区分度和筛选速度一样就按照表中顺序使用第一个索引。如果是联合索引，遵循最左优先，从最左边的索引开始匹配起。

### 20.jvm内存模型
- 程序计数器<br>
线程私有，每条线程都有自己的程序计数器
- java虚拟机栈<br>
虚拟机栈为虚拟机执行java方法服务，方法调用，出栈入栈
- 本地方法栈
本地方法栈则是为虚拟机使用到本地方法服务
- java堆<br>
java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例。
- 方法区<br>
方法区是java堆一样，是各个线程共享的内存区域，他用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码缓存等数据。


### 21.java垃圾回收机制
- 标记清除算法
- 复制算法
- 标记-整理算法
- 分代收集算法




