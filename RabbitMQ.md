# 一、消息队列

## 1、MQ相关概念

### （1）什么是MQ

​		MQ（Message Queue），从字面意思上看，本质是个队列，FIFO先入先出，只不过队列中存放的内容是message而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了MQ之后，消息发送上游只需要依赖MQ，不用依赖其他服务。

### （2）为什么用MQ

#### ①流量消峰

​		举个例子，如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，我们可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。

#### ②应用解耦

​		以电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。

![image-20211210143530606](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210143530606.png)

#### ③异步处理

​		有些服务间调用是异步的，例如 A 调用 B，B 需要花费很长时间执行，但是 A 需要知道 B 什么时候可以执行完，以前一般有两种方式，A 过一段时间去调用 B 的查询 api 查询。或者 A 提供一个 callback api， B 执行完之后调用 api 通知 A 服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题，A 调用 B 服务后，只需要监听 B 处理完成的消息，当 B 处理完成后，会发送一条消息给 MQ，MQ 会将此消息转发给 A 服务。这样 A 服务既不用循环调用 B 的查询 api，也不用提供 callback api。同样 B 服务也不用做这些操作。A 服务还能及时的得到异步处理成功的消息。

![image-20211210143556527](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210143556527.png)

### （3）MQ 的分类

#### ①ActiveMQ

​		优点：单机吞吐量万级，时效性 ms 级，可用性高，基于主从架构实现高可用性，消息可靠性较低的概率丢失数据

​		缺点：官方社区现在对 ActiveMQ 5.x **维护越来越少，高吞吐量场景较少使用**。

#### ②Kafka

​		大数据的杀手锏，谈到大数据领域内的消息传输，则绕不开 Kafka，这款为**大数据而生**的消息中间件，以其**百万级** **TPS** 的吞吐量名声大噪，迅速成为大数据领域的宠儿，在数据采集、传输、存储的过程中发挥着举足轻重的作用。目前已经被 LinkedIn，Uber, Twitter, Netflix 等大公司所采纳。

​		优点: 性能卓越，单机写入 TPS 约在百万条/秒，最大的优点，就是吞**吐量高**。时效性 ms 级可用性非常高，kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用,消费者采用 Pull 方式获取消息, 消息有序, 通过控制能够保证所有消息被消费且仅被消费一次;有优秀的第三方Kafka Web 管理界面 Kafka-Manager；在日志领域比较成熟，被多家公司和多个开源项目使用；功能支持：功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及**日志采集**被大规模使用

​		缺点：Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；支持消息顺序，但是一台代理宕机后，就会产生消息乱序，**社区更新较慢**； 

#### ③RocketMQ

​		RocketMQ 出自阿里巴巴的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进。被阿里巴巴广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog 分发等场景。

​		优点:**单机吞吐量十万级**,可用性非常高，分布式架构,**消息可以做到** **0** **丢失****,**MQ 功能较为完善，还是分布式的，扩展性好,**支持** **10** **亿级别的消息堆积**，不会因为堆积导致性能下降,源码是 java 我们可以自己阅读源码，定制自己公司的 MQ

​		缺点：**支持的客户端语言不多**，目前是 java 及 c++，其中 c++不成熟；社区活跃度一般,没有在 MQ核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码

#### ④RabbitMQ

​		2007 年发布，是一个在 AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是**当前最主流的消息中间件之一**。

​		优点:由于 erlang 语言的**高并发特性**，性能较好；**吞吐量到万级**，MQ 功能比较完备,健壮、稳定、易用、跨平台、**支持多种语言** 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持 AJAX 文档齐全；开源提供的管理界面非常棒，用起来很好用,**社区活跃度高**；更新频率相当高https://www.rabbitmq.com/news.html

​		缺点：商业版需要收费,学习成本较高

### （4）MQ 的选择

#### ①Kafka

​		Kafka 主要特点是基于 Pull 的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，适合产生**大量数据**的互联网服务的数据收集业务。**大型公司**建议可以选用，如果有**日志采集**功能，肯定是首选 kafka 了。

#### ②RocketMQ

​		天生为**金融互联网**领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ 在稳定性上可能更值得信赖，这些业务场景在阿里双 11 已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择 RocketMQ。

#### ③RabbitMQ

​		结合 erlang 语言本身的并发优势，性能好**时效性微秒级**，**社区活跃度也比较高**，管理界面用起来十分方便，如果你的**数据量没有那么大**，中小型公司优先选择功能比较完备的 RabbitMQ。

## 2、RabbitMQ

### （1）RabbitMQ概念

​		RabbitMQ 是一个消息中间件：它接受并转发消息。你可以把它当做一个快递站点，当你要发送一个包裹时，你把你的包裹放到快递站，快递员最终会把你的快递送到收件人那里，按照这种逻辑 RabbitMQ 是一个快递站，一个快递员帮你传递快件。RabbitMQ 与快递站的主要区别在于，它不处理快件而是接收，存储和转发消息数据。

### （2）四大核心概念

- #### 生产者

  ​		产生数据发送消息的程序是生产者

- #### 交换机

  ​		交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

- #### 队列

  ​		队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

- #### 消费者

  ​		消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。

### （3）RabbitMQ 核心部分

![image-20211210144835496](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210144835496.png)

### （4）各个名词介绍

![image-20211210144937824](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210144937824.png)

- **Broker**：接收和分发消息的应用，RabbitMQ Server 就是Message Broker

- **Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

- **Connection**：publisher／consumer 和 broker 之间的 TCP 连接

- **Channel**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的Connection 极大减少了操作系统建立TCP connection 的开销**

- **Exchange**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)

- **Queue**：消息最终被送到这里等待 consumer 取走

- **Binding**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

### （5）安装

#### ①官网地址

https://www.rabbitmq.com/download.html

上传到/usr/local/software 目录下(如果没有 software 需要自己创建) 

```ini
# In /etc/yum.repos.d/rabbitmq.repo

##
## Zero dependency Erlang
##

[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_erlang-source]
name=rabbitmq_erlang-source
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

##
## RabbitMQ server
##

[rabbitmq_server]
name=rabbitmq_server
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_server-source]
name=rabbitmq_server-source
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```



```bash
yum update -y

yum install socat logrotate -y

yum install erlang -y
yum install rabbitmq-server -y
```







#### ③安装文件(顺序安装)

1. rpm -ivh erlang-21.3-1.el7.x86_64.rpm
2. yum install socat -y
3. rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm

#### ④常用命令（顺序执行）

**注意：1.使用阿里云服务器要在安全组配置开放端口**

![image-20211210151448196](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210151448196.png)

​			**2.更改服务器的hostname，以及hosts中IP对应的域名**

```sh
vim /etc/hostname
vim /etc/hosts
```



添加开机启动 RabbitMQ 服务

```sh
chkconfig rabbitmq-server on
```

启动服务

```sh
/sbin/service rabbitmq-server start
```

查看服务状态

```sh
/sbin/service rabbitmq-server status
```

![image-20211210145754220](https://note-1010.oss-cn-beijing.aliyuncs.com/img/image-20211210145754220.png)

停止服务(选择执行)

```sh
/sbin/service rabbitmq-server stop
```

开启 web 管理插件

```sh
rabbitmq-plugins enable rabbitmq_management
```

用默认账号密码(guest)访问地址 http://47.106.196.152:15672/出现权限问题

### （6）新建用户

**创建账号**

```sh
rabbitmqctl add_user admin 123
```

账号admin 密码123

**设置用户角色**

```sh
rabbitmqctl set_user_tags admin administrator
```

设置权限为管理员

**设置用户权限**

```sh
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>

rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限

**当前用户和角色**

```sh
rabbitmqctl list_users
```

### （7）重置命令

关闭应用的命令为

```sh
rabbitmqctl stop_app
```

清除的命令为

```sh
rabbitmqctl reset
```

重新启动命令为

```sh
rabbitmqctl start_app
```



# 二、Hello Word

### （1）导入依赖

```xml
	<!-- jdk编译版本 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <!-- rabbitmq客户端依赖 -->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.8.0</version>
        </dependency>
        <!-- 操作文件io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>
```

### （2）生产者

```java
package pers.cls.rabbitmq.one;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Producer {
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接rabbitmq队列
        factory.setHost("47.106.196.152");
        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("123");
        // 创建连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();
        /**
         * 队列名称
         * 开启持久化
         * 是否独占
         * 是否自动删除
         * 队列其他属性
         *
         */
        channel.queueDeclare(QUEUE_NAME, false, true, true, null);

        String message = "hello world";

        /**
         * 发送消息
         *
         * 交换机
         * 路由的key,队列名称
         * 其他参数信息
         * 发送消息的消息体
         *
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("消息发送完毕");

    }

}

```



### （3）消费者

```java
package pers.cls.rabbitmq.one;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.Arrays;
import java.util.concurrent.TimeoutException;

public class Consumer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接rabbitmq队列
        factory.setHost("47.106.196.152");
        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("123");
        // 创建连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();


        // 接收消息之后的回调
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(Arrays.toString(message.getBody()));
        };

        // 消息接收取消后的回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息已取消接收");
        };

        /**
         * 队列名称
         * 消费者未成功消费的回调
         * 消费者取录消费的回调
         */

        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }

}

```



### （4）处理报错

**连接超时**：**5672端口放行**

```base
Exception in thread "main" java.net.ConnectException: Connection timed out: connect
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:81)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:476)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:218)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:200)
	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:162)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:394)
	at java.net.Socket.connect(Socket.java:606)
	at com.rabbitmq.client.impl.SocketFrameHandlerFactory.create(SocketFrameHandlerFactory.java:60)
	at com.rabbitmq.client.impl.recovery.RecoveryAwareAMQConnectionFactory.newConnection(RecoveryAwareAMQConnectionFactory.java:62)
	at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.init(AutorecoveringConnection.java:156)
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1110)
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1067)
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1025)
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1187)
	at pers.cls.rabbitmq.one.Producer.main(Producer.java:23)
```



生产者信道 exclusive 参数如果为true，消费者信道应有对应参数为true

报错

关键信息 cannot obtain exclusive access to locked queue 'hello' in vhost '/'.

```
Exception in thread "main" java.io.IOException
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:129)
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:125)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1384)
	at com.rabbitmq.client.impl.recovery.AutorecoveringChannel.basicConsume(AutorecoveringChannel.java:532)
	at com.rabbitmq.client.impl.recovery.AutorecoveringChannel.basicConsume(AutorecoveringChannel.java:486)
	at com.rabbitmq.client.impl.recovery.AutorecoveringChannel.basicConsume(AutorecoveringChannel.java:469)
	at pers.cls.rabbitmq.one.Consumer.main(Consumer.java:44)
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=405, reply-text=RESOURCE_LOCKED - cannot obtain exclusive access to locked queue 'hello' in vhost '/'. It could be originally declared on another connection or the exclusive property value does not match that of the original declaration., class-id=60, method-id=20)
	at com.rabbitmq.utility.ValueOrException.getValue(ValueOrException.java:66)
	at com.rabbitmq.utility.BlockingValueOrException.uninterruptibleGetValue(BlockingValueOrException.java:36)
	at com.rabbitmq.client.impl.AMQChannel$BlockingRpcContinuation.getReply(AMQChannel.java:502)
	at com.rabbitmq.client.impl.ChannelN.basicConsume(ChannelN.java:1378)
	... 4 more
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=405, reply-text=RESOURCE_LOCKED - cannot obtain exclusive access to locked queue 'hello' in vhost '/'. It could be originally declared on another connection or the exclusive property value does not match that of the original declaration., class-id=60, method-id=20)
	at com.rabbitmq.client.impl.ChannelN.asyncShutdown(ChannelN.java:522)
	at com.rabbitmq.client.impl.ChannelN.processAsync(ChannelN.java:346)
	at com.rabbitmq.client.impl.AMQChannel.handleCompleteInboundCommand(AMQChannel.java:182)
	at com.rabbitmq.client.impl.AMQChannel.handleFrame(AMQChannel.java:114)
	at com.rabbitmq.client.impl.AMQConnection.readFrame(AMQConnection.java:719)
	at com.rabbitmq.client.impl.AMQConnection.access$300(AMQConnection.java:48)
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:646)
	at java.lang.Thread.run(Thread.java:748)

```

# 三、Work Queues

![image-20220308161140676](F:/note/RabbitMQ.assets/image-20220308161140676.png)

​		工作队列（任务队列）的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程及那个弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。



## 1、轮训分发消息





## 2、消息应答

### （1）概念

​		消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。
