---
title: RabbitMq
date: 2019-06-27 23:32:33
tags:
- RabbitMq
categories: 
- Plugs/RabbitMq
---
### 消息中间件
+ 消息中间件是指利用高效可靠得消息传递机制进行与平台数据无关的数据交流，并基于数据通信来进行分布式系统集成。
+ RabbitMq整体上是一个生产者和消费者模型，主要负责接收，存储和转发消息。
+ RabbitMq就是AMQP(应用层高级消息队列协议)协议的Erlang的实现。

### 作用 
1. 解耦：独立修改或者扩展两边的处理过程，只要确保遵循相同的接口约束即可。
2. 削峰：使用中间件能够使关键组件支撑突发访问压力，不会因为突发的超负荷请求而完全崩溃。
3. 异步通信：很多时候不想立即处理消息，中间件允许把消息放到中间件中，但是不立即处理它，之后需要的时候在慢慢处理。

### 名词解释
1. 生产者：投递消息的一方.
2. 消费者: 接收消息的一方.
3. Broker：消息中间件的服务节点.
4. Queue: 队列，RabbitMq的内部对象.
5. 交换器：生产者把消息发送到Exchange（交换器）,由交换器把消息路由到一个或者多个队列中,如果路由不到,或许会返回给生产者,或许会直接丢弃.RabbitMq有四种交换机类型:[交换机类型](https://www.cnblogs.com/WJ--NET/p/8203380.html)

### 五种模式
1. 简单模式：一个生产者，一个消费者.
    >简单模式特点，其实就是是生成者直接将消息发送给队列，消费者直接从队列取消息，中间没有其他的东西，并且1对1的。

    - 生产者
``` csharp
     //创建mq的连接工厂
     var factory = new ConnectionFactory();
     factory.HostName = "119.29.225.20";
     factory.UserName = "userId";
     factory.Password = "pwd";
     //创建一个连接
     var connection = factory.CreateConnection();
     //创建一个频道
     var channel = connection.CreateModel();
     //创建一个队列
     // durable:持久化可以把交换器存盘，在服务器重启的时候不会丢失相关信息.
     // exclusive:如果申明为排它队列,该队列仅对首次申明它的连接可见,并且在连接断开的时候自动删除,
     // autoDelete:是否自动删除,自动删除的前提是：至少又一个消费者连接到这个队列,
     // 之后所有与这个队列连接的消费者都断开时,才会自动删除.
     channel.QueueDeclare(queue: "hello",
                          durable: false,
                          exclusive: false,
                          autoDelete: false,
                          arguments: null);
     for (int i = 0; i < 50; i++)
     {
         string message = "Hello World!"+i;
         var body = Encoding.UTF8.GetBytes(message) 
         // 发送消息
         channel.BasicPublish(exchange: "",
                              routingKey: "hello",
                              basicProperties: null,
                              body: body);
         Console.WriteLine(" [x] Sent {0}", message);
     }
```
    - 消费者
``` csharp
        var factory = new ConnectionFactory();
        factory.HostName = "119.29.225.20";
        factory.UserName = "userId";
        factory.Password = "pwd";
        using (var connection = factory.CreateConnection())
        {
            using (var channel = connection.CreateModel())
            {
                channel.QueueDeclare(queue: "hello",
                                     durable: false,
                                     exclusive: false,
                                     autoDelete: false,
                                     arguments: null);
                var consumer = new EventingBasicConsumer(channel);
                consumer.Received += (model, ea) =>
                {
                    var body = ea.Body;
                    var message = Encoding.UTF8.GetString(body);
                    Console.WriteLine(" [x] Received {0}", message);
                };
                channel.BasicConsume(queue: "hello",
                                     autoAck: true,
                                     consumer: consumer);
                Console.ReadKey();
            }
        }
```

2. 工作模式：一个生产者,多个消费者.
    >消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2,同时监听同一个队列,消息被消费?C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患,高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize,与同步锁的性能不一样) 保证一条消息只能被一个消费者使用).
    >一条消息只会被一个消费者接收；rabbit采用轮询的方式将消息是平均发送给消费者的；消费者在处理完某条消息后，才会收到下一条消息.
```csharp

```



