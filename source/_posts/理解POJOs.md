---
title: 理解POJO
date: 2017-12-14 22:43:00
categories:
- 编程语言
- Java
tags:
- Java
---


POJO表示Plain Old Java Object。它是一个java对象的实例，并且不耦合在各种框架扩展中。

比如，想从JMS中取出消息，你需要编写一个类实现`MessageListener`接口。

```java
public class ExampleListener implements MessageListener {

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                System.out.println(((TextMessage) message).getText());
            }
            catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        }
        else {
            throw new IllegalArgumentException("Message must be of type TextMessage");
        }
    }

}
```

这会使你的代码变得不通用，迁移到其他消息中间件实现时会变的困难。如果你的应用使用了大量的监听器，

那么基于以上的情形选择AMQP或其它方案将变得几乎不可能。

基于POJO的实现意味着你的消息处理不需实现具体框架的接口。

```java
@Component
public class ExampleListener {

    @JmsListener(destination = "myDestination")
    public void processOrder(String message) {
    	System.out.println(message);
    }
}
```

在这个例子中，你的代码没有直接绑定任何接口。取而代之的是，连接JMS队列的责任被转移到了

注解中，并且注解更容易更新。当前示例中，你可以用@RabbitListener替换@JmsListener。在其他

情形下，基于POJO的实现方案可能不使用任何注解。

这只是一个小例子，它没有对比JMS和Rabbit MQ，而是用以说明代码不绑定接口的意义。通过使用POJO，

你的代码变得更简单。这样有助于更好的测试，灵活性以及应对以后发生改变的情况。

Spring及各种组件始终致力于减少代码和类库之间的耦合。这是依赖注入的首要概念，

即你的服务(指框架组件等)被使用的方式应该是接通应用程序的一部分，而不是服务本身(否则应用和服务发生耦合)。

https://spring.io/understanding/POJO













