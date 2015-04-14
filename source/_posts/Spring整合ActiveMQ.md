title: "Spring整合ActiveMQ"
date: 2015-04-13 16:30:59
categories: java
tags: [jms,activemq]
---

##JMS
JMS的全称是Java Message Service，即Java消息服务。
它主要用于在生产者和消费者之间进行消息传递，生产者负责产生消息，而消费者负责接收消息。
把它应用到实际的业务需求中的话我们可以在特定的时候利用生产者生成一消息，并进行发送，对应的消费者在接收到对应的消息后去完成对应的业务逻辑。
对于消息的传递有两种类型，一种是点对点的，即一个生产者和一个消费者一一对应；另一种是发布/订阅模式，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。
JMS只是一个规范的定义，实际的实现还要依靠相关的JMS provider 这里以activeMQ为例。

---

##配置
###配置connectionFactory
ConnectionFactory是用于产生到JMS服务器的链接的，Spring为我们提供了多个ConnectionFactory，有SingleConnectionFactory和CachingConnectionFactory。
SingleConnectionFactory对于建立JMS服务器链接的请求会一直返回同一个链接，并且会忽略Connection的close方法调用。
CachingConnectionFactory继承了SingleConnectionFactory，所以它拥有SingleConnectionFactory的所有功能，同时它还新增了缓存功能，它可以缓存Session、MessageProducer和MessageConsumer。
简单配置如下：
```
    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供-->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616"/>
    </bean>
    
    <bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">
        <property name="connectionFactory" ref="targetConnectionFactory"/>
        <property name="maxConnections" value="10"/>
    </bean>
    
    <bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
        <property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
    </bean>
    
```
PooledConnectionFactory，通过往里面注入一个ActiveMQConnectionFactory可以用来将Connection、Session和MessageProducer池化，这样可以大大的减少我们的资源消耗。

###配置生产者
生产者负责产生消息并发送到JMS服务器，这通常对应的是我们的一个业务逻辑服务实现类。但是我们的服务实现类是怎么进行消息的发送的呢？这通常是利用Spring为我们提供的JmsTemplate类来实现的，`所以配置生产者其实最核心的就是配置进行消息发送的JmsTemplate`。对于消息发送者而言，它在发送消息的时候要知道自己该往哪里发，为此，我们在定义JmsTemplate的时候需要往里面注入一个Spring提供的ConnectionFactory对象。

```
    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
        <property name="connectionFactory" ref="connectionFactory"/>
    </bean>
```
在真正利用JmsTemplate进行消息发送的时候，我们需要知道消息发送的目的地，即destination。
在Jms中有一个用来表示目的地的Destination接口，它里面没有任何方法定义，只是用来做一个标识而已。
当我们在使用JmsTemplate进行消息发送时没有指定destination的时候将使用默认的Destination。
默认Destination可以通过在定义jmsTemplate bean对象时通过属性defaultDestination或defaultDestinationName来进行注入，defaultDestinationName对应的就是一个普通字符串。
>在ActiveMQ中实现了两种类型的Destination，一个是点对点的ActiveMQQueue，另一个就是支持订阅/发布模式的ActiveMQTopic。

在定义这两种类型的Destination时我们都可以通过一个name属性来进行构造，如：

```
    <!--这个是队列目的地，点对点的-->
    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg>
            <value>queue</value>
        </constructor-arg>
    </bean>
    <!--这个是主题目的地，一对多的-->
    <bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="topic"/>
    </bean>

```
这样在调用jmsTemplate的时候就能默认往配置的队列中发了

###配置消费者
生产者往指定目的地Destination发送消息后，接下来就是消费者对指定目的地的消息进行消费了。那么消费者是如何知道有生产者发送消息到指定目的地Destination了呢？
这是通过Spring为我们封装的消息监听容器MessageListenerContainer实现的，它负责接收信息，并把接收到的信息分发给真正的MessageListener进行处理。每个消费者对应每个目的地都需要有对应的MessageListenerContainer。对于消息监听容器而言，除了要知道监听哪个目的地之外，还需要知道到哪里去监听，也就是说它还需要知道去监听哪个JMS服务器，这是通过在配置MessageConnectionFactory的时候往里面注入一个ConnectionFactory来实现的。

<font color="red">所以我们在配置一个MessageListenerContainer的时候有三个属性必须指定，一个是表示从哪里监听的ConnectionFactory；一个是表示监听什么的Destination；一个是接收到消息以后进行消息处理的MessageListener。</font>

Spring一共为我们提供了两种类型的MessageListenerContainer，SimpleMessageListenerContainer和DefaultMessageListenerContainer。

SimpleMessageListenerContainer会在一开始的时候就创建一个会话session和消费者Consumer，并且会使用标准的JMS MessageConsumer.setMessageListener()方法注册监听器让JMS提供者调用监听器的回调函数。它不会动态的适应运行时需要和参与外部的事务管理。兼容性方面，它非常接近于独立的JMS规范，但一般不兼容Java EE的JMS限制。

大多数情况下我们还是使用的DefaultMessageListenerContainer，跟SimpleMessageListenerContainer相比，DefaultMessageListenerContainer会动态的适应运行时需要，并且能够参与外部的事务管理。它很好的平衡了对JMS提供者要求低、先进功能如事务参与和兼容Java EE环境。
定义处理消息的MessageListener
```
    <!--这个是队列目的地-->
    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg>
            <value>queue</value>
        </constructor-arg>
    </bean>
    <!-- 消息监听器 自己定义的继承MessageListener接口的类-->
    <bean id="consumerMessageListener" class="com.xxx.ConsumerMessageListener"/>    

    <!-- 消息监听容器 -->
    <bean id="jmsContainer"        class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="queueDestination" />
        <property name="messageListener" ref="consumerMessageListener" />
    </bean>

```

>一般要定义处理消息的MessageListener我们只需要实现JMS规范中的MessageListener接口就可以了。MessageListener接口中只有一个方法onMessage方法，当接收到消息的时候会自动调用该方法。可以在该方法中定义处理的逻辑。

---

>整个流程是这样子的：业务逻辑调用jmsTempla中的send()方法发送消息到服务器的相关的queue，然后MessageListenerContainer监听队列中的消息，当有消息到达时候将回调其中注册的MessageListener中的onMessage()业务逻辑

当然这前面只是简单的介绍如何实现一个消费者和生产者之间的通信。
