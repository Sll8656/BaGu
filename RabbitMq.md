## 使用

publicser `->exchange ->queue-> `consumer

topicExchange通过routingkey和queue进行绑定，发送消息的时候就给他指定routingkey和topicExchange，然后topicExchange根据routingkey将这个消息发送到通过routingkey绑定的队列。

publicsher只用routingkey发送到交换机，与队列无关。consumer只和队列通信，与routingkey和交换机无关。

---

1. 交换机将消息路由到多个队列：
   交换机（Exchange） 可以根据其类型（如 Direct、Topic、Fanout 等）以及消息的路由键，将一条消息路由到一个或多个队列。
   如果有多个队列绑定到同一个交换机，并且它们符合消息的路由规则（例如使用相同的路由键或匹配的路由模式），那么**==这条消息会被复制并发送==**到所有符合条件的队列。

2. 队列中的消息只能被消费一次：
   队列（Queue） 是一个消息的存储容器，每条消息只能在队列中被消费一次。一旦消费者成功消费了消息，这条消息会从队列中删除，不再提供给其他消费者。
   如果有多个消费者同时监听同一个队列，每条消息只会被其中一个消费者消费。这是 RabbitMQ 保证消息的至少一次消费（At-least-once Delivery）的方式。

---

**@RabbitListener** 可以同时监听多个队列。可以通过配置 @RabbitListener 注解的 queues 属性来指定一个或多个队列。当这些队列中的任何一个接收到消息时，@RabbitListener 标注的方法都会被触发来处理消息。

**@RabbitListener(queues = {"queue1", "queue2"})**：这里指定了两个队列 queue1 和 queue2。当这两个队列中的任意一个收到消息时，方法都会被触发。

---

## 模板代码：

```java
//消息发送
public void send() {
  //...
  rabbitTemplate.convertAndSend(
                "交换机名称",
                "routingKey",
                "消息内容（封装类型）");
}
// 监听事件
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "队列名", durable = "true"),
        exchange = @Exchange(name = "交换机名称", type = ExchangeTypes.TOPIC),
        key = "routingKey"
))
public void listenSignInMessage(Object message){
} 
```

解释：

1. 监听事件的@RabbitListener是负责==声明==交换机、routingkey、队列的。并且声明完成后，自己会监听这个==队列==。
2. 发送消息，指定交换机、routingKey、消息内容。其中消息内容可以是封装的实体类。接收的时候也用这个实体类接收即可

---

mq是异步的，rpc是同步的。

mq通常来说不会在同一个模块,但是也可以在一个模块。

注意：如果使用消息转换器序列化实体类，则：

1. 实体类需要提供构造方法，
2. publisher和listener都要配置消息转换器，使用jackson的



## 消息可靠性

> **如何保证消息可靠性？**

![image-20240920145221843](https://s2.loli.net/2024/09/20/rQHPpKXOUsivY5h.png)

三个层面：

1. 消息没有到达 mq：生产者确认（confirm）机制，mq收到消息会给publisher发送ack。
2. 消息在mq中丢失：队列持久化、交换机持久化、消息持久化
3. 消费者没接受到消息：
   1. 消费者确认机制，消费者消费完消息给mq发送确认。
   2. 失败重试机制：消费出现异常会重试，次数到达上限后，消息被投递进异常交换机，由人工处理。

## 消息有序性

消息有序指的是可以按照消息的发送顺序来消费。假如生产者产生了2 条消息：M1、M2，假定 M1 发送到 S1，M2 发送到 S2，如果要保证 M1 先于 M2 被消费，怎么做？

![image-20240925163832290](https://s2.loli.net/2024/09/25/9PEIcVxUDW7ta2Q.png)

**解决方案**：

保证生产者 - MQServer - 消费者是一对一对一的关系

![image-20240925163850379](https://s2.loli.net/2024/09/25/TBfn145bMh6JqaV.png)

并行度就会成为消息系统的瓶颈（吞吐量不够）
更多的异常处理，比如：只要消费端出现问题，就会导致整个处理流程阻塞，我们不得不花费更多的精力来解决阻塞的问题。
通过合理的设计或者将问题分解来规避。
不关注乱序的应用实际大量存在
队列无序并不意味着消息无序 所以从业务层面来保证消息的顺序而不仅仅是依赖于消息系统，是一种更合理的方式。

## 消息重复消费

> **如何保证消息不被重复消费？**

幂等性设计：

**使用唯一标识符**：为每条消息分配一个唯一的 ID（如 UUID），消费者在处理消息前检查该 ID 是否已处理过。如果已处理，则跳过处理。

**记录已处理的消息**：使用数据库或缓存（如 Redis）存储已处理的消息 ID。处理前先查询记录，确保不重复处理。







## 死信交换机（延迟队列）

> 说说mq的死信交换机

**延迟队列的场景**：超时订单自动取消、限时优惠、内容审核、定时发布（选时间发朋友圈）

**==延迟队列 = 死信交换机 + ttl==**

![image-20240920152405328](https://s2.loli.net/2024/09/20/LVgcN9DAbUi4B5S.png)

普通队列中的消息超时 ->进入死信交换机，然后消费死信交换机的即可。



## 消息堆积

> 消息堆积怎么解决

1. 增加消费者的数量

2. 提高消费者的消费效率：使用线程池

3. 限制生产者的速率

4. 扩大队列容积：使用惰性队列（接收到消息直接存入磁盘）

   ```java
   public Queue lazeQueue() {
     return QueueBuilder.durable("lazy.queue").lazy().build();
   }
   ```



## 高可用

> 如何保证rabbitmq的高可用

使用镜像队列：镜像队列类似于redis的集群，一主多从。主宕机后，镜像结点会替代成新的主。



## Channel 概念

​	信道是生产消费者与rabbit通信的渠道，生产者publish或者消费者消费一个队列都是需要通过信道来通信的。信道是建立在TCP上面的虚拟链接，也就是rabbitMQ在一个TCP上面建立成百上千的信道来达到多个线程处理。**注意是一个TCP 被多个线程共享，每个线程对应一个信道，信道在rabbit都有唯一的ID，保证了信道的私有性，对应上唯一的线程使用。**

==为什么RabbitMQ 需要信道，如果直接进行TCP通信呢？==

关键点：复用TCP连接

上述的描述其实已经很明显了，因为TCP可以被多个线程共享，显然线程比TCP要省事的多。

TCP的创建开销很大，创建需要三次握手，销毁需要四次握手。

如果不使用信道，那么引用程序就会使用TCP方式进行连接到RabbitMQ，因为MQ可能每秒会进行成千上万的链接，

总之就是TCP消耗资源，TCP链接可以容纳无限的信道，不会有并发上面的性能瓶颈。



在代码中并不会有直观的能看到信道这个概念。

因为代码中都是用自动配置。

```java
@Autowired
private RabbitTemplate rabbitTemplate; 
或者
@Autowired
private AmqpTemplate template;
```

都是自动隐藏了详细的建立连接过程。

但是在使用rabbitmq时不管是消费还是生产都需要创建信道（channel） 和connection（连接）。

连接是连接到RabbitMQ的服务器。

示意图：

![image-20240925173656193](https://s2.loli.net/2024/09/25/eh9xUM426RyFrHS.png)

看这个示意图的话，我们好像可以直接使用Connection就可以完成信道的工作。

但是因为建立连接会很耗费性能，这也是有点类似于工厂模式那种吧~用类似nio的做法，tcp连接复用，当每个信道的流量不是很大时，复用。但是当流量很大的时候，多个信道用一个connection就会出现性能的瓶颈。所以使用多个connection也是合理的，这样信道可以平摊到每个connection中。具体调节方式需要在业务中进行体验。

