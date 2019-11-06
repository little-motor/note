[toc]
# 1. 引言
rabbit mq常用基础概念知识
# 2. 启动服务
```
# 后台运行，服务默认端口号5672
rabbitmq-server -detached
# 查看集群信息
rabbitmqctl cluster_status
# 新增用户
rabbitmqctl add_user username password
# 设置用户权限
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
# 设置管理员角色
rabbitmqctl set_user_tags username administrator
# 启用web界面，服务默认端口号15672
rabbitmq-plugins enable rabbitmq_management
# 获取插件列表
rabbitmq-plugins list
```
# 3. 相关概念
## 3.1 整体模型
![rabbitMq](https://raw.githubusercontent.com/little-motor/uml/master/distributedSystem/mq/rabbitmq%E6%A8%A1%E5%9E%8B%E6%9E%B6%E6%9E%84.png)
## 3.2 生产者和消费者
- 生产者：生产消息然后发送(Basic.Publish)到RabbitMQ中，消息一般包含两个部分：消息体（payload）和标签（Label）
- 消费者：消息接收方，只消费消息(Basic.Consume/Basic.Get)的消息体（payload），消息路由的过程中，消息的标签会丢弃。
- Broker：消息中间件的服务节点，可以简单的看作一个RabbitMQ服务节点。

![rabbitMq](https://raw.githubusercontent.com/little-motor/uml/master/distributedSystem/mq/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E8%BF%90%E8%BD%AC%E8%BF%87%E7%A8%8B.png)
## 3.3 队列
RabbitMQ中消息都只能存储在队列中，多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊。队列跟交换器共享某些属性，但是队列也有一些另外的属性。
- Name
- Durable（消息代理重启后，队列依旧存在）
- Exclusive（只被一个连接（connection）使用，而且当连接关闭后队列即被删除）
-  Auto-delete（当最后一个消费者退订后即被删除）
-  Arguments（一些消息代理用他来完成类似与 TTL 的某些额外功能）
## 3.4 绑定键、路由键、交换器
### 3.4.1 绑定键
BindingKey：绑定键。RabbitMQ中通过绑定将交换器与队列关联起来，生产者发送消息给交换器时，需要一个RoutingKey，当传入消息的RoutingKey与BindingKey相匹配时，消息会被路由到对应的队列中。
### 3.4.2 路由键
RoutingKey：路由键。用来指定这个消息的路由规则。其实BindingKey也是路由键的一种，大多数时候官方文档和RabbitMQ API都把BindingKey和RoutingKey看作RoutingKey，可以这样理解在绑定的时候路由键是BindingKey，如：channel.exchangeBind,channel.queueBind。在发送消息的时候，其中需要的路由键是RoutingKey，涉及的客户端方法如：channel.basciPublish。

在topic交换器类型下，RoutingKey和BindingKey之间需要进行模糊匹配，两者是不同的。简而言之BingdingKey是位于交换器上的RoutingKey，位于生产者发送的消息中的RoutingKey将会与交换器的BindingKey匹配，并根据匹配结果放入相应队列。
### 3.4.3 交换器类型
常用的交换器类型有四种：fanout、direct、topic、headers
- fanout：会把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。
- direct：会把消息路由到RoutingKey和BindingKey完全匹配的队列中。
  ![rabbitMq](https://raw.githubusercontent.com/little-motor/uml/master/distributedSystem/mq/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E8%BF%90%E8%BD%AC%E8%BF%87%E7%A8%8B.png)
- topic： 类似正则匹配，他规定：
  1. RoutingKey为一个点号"."分隔的字符串，如"com.rabbitmq.client"
  2. BindingKey也是用点号"."分隔的字符串，但是可以存在两种特殊字符"\*"和"#"，"\*"用于匹配一个单词，"#"用于匹配0-n个单词和"."
  ![rabbitMq](https://raw.githubusercontent.com/little-motor/uml/master/distributedSystem/mq/topic%E7%B1%BB%E5%9E%8B%E4%BA%A4%E6%8D%A2%E5%99%A8.png)
- headers：根据消息内容中的headers属性进行匹配，性能较差。
# 4. RabbitMQ运转流程
## 4.1 生产者发送消息
1. 生产者连接到RabbitMQ Broker ， 建立一个连接( Connection) ，开启一个信道(Channel)
2. 生产者声明一个交换器，并设置相关属性，比如交换机类型、是否持久化等
3. 生产者声明一个队列井设置相关属性，比如是否排他、是否持久化、是否自动删除等
4. 生产者通过路由键将交换器和队列绑定起来
5. 生产者发送消息至RabbitMQ Broker，其中包含路由键、交换器等信息
6. 相应的交换器根据接收到的路由键查找相匹配的队列
7. 如果找到，则将从生产者发送过来的消息存入相应的队列中
8. 如果没有找到，则根据生产者配置的属性选择丢弃还是回退给生产者
9. 关闭信道
10. 关闭连接
## 4.2 消费者接收消息的过程:
1. 消费者连接到RabbitMQ Broker，建立一个连接(Connection)，开启一个信道(Channel)
2. 消费者向RabbitMQ Broker请求消费相应队列中的消息，可能会设置相应的回调函数，
以及做一些准备工作
3. 等待RabbitMQ Broker 回应并投递相应队列中的消息，消费者接收消息
4. 消费者确认(ack) 接收到的消息
5. RabbitMQ 从队列中删除相应己经被确认的消息
6. 关闭信道
7. 关闭连接