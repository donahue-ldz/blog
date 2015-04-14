title: "Spring JMS 三种监听器"
date: 2015-04-14 19:25:48
categories: java
tags: Jms,ActiveMQ
---

前一文中说了简单的JMS的结构，选择了一种监听器的结构，但是真的只有一种监听器吗？
本文讲述的是三种监听器，用来消费生产者发送的Destination的消息

##消息监听器
###MessageListener
MessageListener是最原始的消息监听器，它是JMS规范中定义的一个接口。
其中定义了一个用于处理接收到的消息的onMessage方法，该方法只接收一个Message参数。
我们前面在讲配置消费者的时候用的消息监听器就是MessageListener，此处不再赘述！但是我们这就很快发现有一个弊端就是，当消息监听者接收到这个消息的时候，如何告诉发送者，我已经接收到这个消息呢？

---
###SessionAwareMessageListener

**SessionAwareMessageListener是Spring为我们提供的，它不是标准的JMS MessageListener**.
MessageListener的设计只是纯粹用来接收消息的，假如我们在使用MessageListener处理接收到的消息时我们需要发送一个消息通知对方我们已经收到这个消息了，那么这个时候我们就需要在代码里面去重新获取一个Connection或Session。SessionAwareMessageListener的设计就是为了方便我们在接收到消息后发送一个回复的消息，它同样为我们提供了一个处理接收到的消息的onMessage方法，但是这个方法可以同时接收两个参数，一个是表示当前接收到的消息Message，另一个就是可以用来发送消息的Session对象,我们就直接利用该Session对象创建相关对象发送就可以了。

---

### MessageListenerAdapter
MessageListenerAdapter类实现了MessageListener接口和SessionAwareMessageListener接口，它的主要作用是将接收到的消息进行类型转换，然后通过反射的形式把它交给一个`普通的Java类进行处理`。MessageListenerAdapter会把接收到的消息做如下转换：
>TextMessage转换为String对象；
BytesMessage转换为byte数组；
MapMessage转换为Map对象；
ObjectMessage转换为对应的Serializable对象。

既然前面说了MessageListenerAdapter会把接收到的消息做一个类型转换，然后利用反射把它交给真正的目标处理器——一个普通的Java类进行处理（如果真正的目标处理器是一个MessageListener或者是一个SessionAwareMessageListener，那么Spring将直接使用接收到的Message对象作为参数调用它们的onMessage方法，而不会再利用反射去进行调用），那么我们在定义一个MessageListenerAdapter的时候就需要为它指定这样一个目标类。这个目标类我们可以通过MessageListenerAdapter的构造方法参数指定，如
```
<!-- 消息监听适配器 -->
    <bean id="messageListenerAdapter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
        <property name="delegate">
            <bean class="com.xxx.ConsumerListener"/>
        </property>
        <property name="defaultListenerMethod" value="receiveMessage"/>
    </bean>
```
这样进行配置的时候在消息来的时候，将自动调用delegate指定的类的和defaultListenerMethod指定的方法
defaultListenerMethod当我们没有指定该属性时，Spring会默认调用目标处理器的handleMessage方法

<font color = "red">MessageListenerAdapter除了会自动的把一个普通Java类当做MessageListener来处理接收到的消息之外，其另外一个主要的功能是可以自动的发送返回消息。</font>
     当我们用于处理接收到的消息的方法的返回值不为空的时候，Spring会自动将它封装为一个JMS Message，然后自动进行回复。那么这个时候这个回复消息将发送到哪里呢？这主要有两种方式可以指定。
 * 可以通过发送的Message的setJMSReplyTo方法指定该消息对应的回复消息的目的地。这里我们把我们的生产者发送消息的代码做一下修改，在发送消息之前先指定该消息对应的回复目的地为一个叫responseQueue的队列目的地
 * 通过MessageListenerAdapter的defaultResponseDestination属性来指定。
 
MessageListenerAdapter会自动把真正的消息处理器返回的非空内容封装成一个Message发送回复消息到通过defaultResponseDestination属性指定的默认消息回复目的地。当两种方法都指定的时候，setJMSReplyTo指定的队列的优先级将要高点

