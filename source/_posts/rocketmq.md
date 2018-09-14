---
title: rocketmq
date: 2018-07-17 00:03:19
tags: rocketmq
categories: Java
top: 100
---
[quick-start](https://rocketmq.apache.org/docs/quick-start/)

### 物理部署

![物理部署](http://img3.tbcdn.cn/5476e8b07b923/TB18GKUPXXXXXXRXFXXXXXXXXXX)

- Name Server是一个**几乎无状态**节点，可集群部署，**节点之间无任何信息同步**。
- Broker分为Master与Slave(**主从配置**)，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定**相同的BrokerName**，不同的BrokerId来定义，**BrokerId为0表示Master，非0表示Slave**。Master也可以部署多个。每个Broker与Name Server集群中的所有节点建立长连接，定时**注册Topic信息到所有Name Server**。
- Producer与Name Server集群中的其中**一个节点（随机选择）建立长连接**，定期从Name Server**取Topic路由信息**，并向提供Topic服务的**Master建立长连接**，且定时向Master发送心跳。**Producer完全无状态**，可集群部署
- Consumer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。

### 编译安装

4.2.0版本：
- git clone https://github.com/apache/rocketmq.git
- 编译 mvn -Prelease-all -DskipTests clean install -U
旧版本：
- git clone https://github.com/apache/incubator-rocketmq.git
- mvn clean package install -Prelease-all assembly:assembly -U

注意安装要求，maven是3.2.X否则可能会有问题(我的3.5.3运行quickstart的Consumer就有问题)

启动关闭:
cd distribution/target/apache-rocketmq

- nohup sh bin/mqnamesrv &
- nohup sh bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true &,不加**autoCreateTopicEnable=true**我这边启动Producer报No route info of this topic错
- 使用tail -f ~/logs/rocketmqlogs/XXX.log或jps查看是否启动成功，Ctrl c退出
- 使用sh bin/mqshutdown broker和sh bin/mqshutdown namesrv关闭

### jar包依赖

[maven](https://mvnrepository.com/artifact/org.apache.rocketmq)

```
commons-cli-1.2.jar
commons-lang3-3.4.jar
fastjson-1.2.29.jar
guava-19.0.jar
javassist-3.20.0-GA.jar
jna-4.2.2.jar
logback-classic-1.0.13.jar
logback-core-1.0.13.jar
netty-all-4.0.36.Final.jar
openmessaging-api-0.1.0-alpha.jar
rocketmq-client-4.2.0-incubating-SNAPSHOT.jar
rocketmq-common-4.2.0-incubating-SNAPSHOT.jar
rocketmq-remoting-4.2.0-incubating-SNAPSHOT.jar
rocketmq-srvutil-4.2.0-incubating-SNAPSHOT.jar
rocketmq-tools-4.2.0-incubating-SNAPSHOT.jar
slf4j-api-1.7.5.jar
```

### 三种方式
- 同步——用在大规模分布场景下，像重要的通知消息，短信通知，短信市场系统等

```
public class SyncProducer {
    public static void main(String[] args) throws Exception {
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new
            DefaultMQProducer("please_rename_unique_group_name");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                "TagA" /* Tag */,
                ("Hello RocketMQ " +
                    i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}
```

- 异步——实时的业务场景

可以看出多了发送完的回调函数

```
public class AsyncProducer {
    public static void main(String[] args) throws Exception {
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
        //Launch the instance.
        producer.start();
        producer.setRetryTimesWhenSendAsyncFailed(0);
        for (int i = 0; i < 100; i++) {
                final int index = i;
                //Create a message instance, specifying topic, tag and message body.
                Message msg = new Message("TopicTest",
                    "TagA",
                    "OrderID188",
                    "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
                producer.send(msg, new SendCallback() {
                    @Override
                    public void onSuccess(SendResult sendResult) {
                        System.out.printf("%-10d OK %s %n", index,
                            sendResult.getMsgId());
                    }
                    @Override
                    public void onException(Throwable e) {
                        System.out.printf("%-10d Exception %s %n", index, e);
                        e.printStackTrace();
                    }
                });
        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}
```

- 单向传输——对可靠性要求一般的场景，像日志搜集

```
public class OnewayProducer {
    public static void main(String[] args) throws Exception{
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                "TagA" /* Tag */,
                ("Hello RocketMQ " +
                    i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            producer.sendOneway(msg);

        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}
```

### 逻辑架构

 ![](http://img3.tbcdn.cn/5476e8b07b923/TB1rdyvPXXXXXcBapXXXXXXXXXX)

- 队列集合称为Topic，Consumer如果做广播消费，则一个consumer实例消费这个Topic对应的所有队列，如果做集群消费，则多个Consumer实例平均消费这个topic对应的队列集合。

#### 有序消息
##### 生产者
从运行结果看，默认四条消息队列(queueid0-3)。消息的id(0~9)会发送到不同的消息队列上

```
    public static void main( String[] args ) throws Exception
    {
        //Instantiate with a producer group name.
        MQProducer producer = new DefaultMQProducer("example_group_name");
        //Launch the instance.
        //多个地址用分号隔开；也可以设置成环境变量
        ((DefaultMQProducer) producer).setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        String[] tags = new String[] {"TagA", "TagB", "TagC", "TagD", "TagE"};
        for (int i = 0; i < 100; i++) {
            int orderId = i % 10;
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTestjjj", tags[i % tags.length], "KEY" + i,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                    Integer id = (Integer) arg;
                    //输出0~9
                    //System.out.println(id);
                    int index = id % mqs.size();// 如果队列数不变，同一个订单号取到的队列是同一个 
                    return mqs.get(index);
                }
            }, orderId);//orderId传递给回调方法

            System.out.printf("%s%n", sendResult);
        }
        //server shutdown
        producer.shutdown();
    }
```

##### 消费者

![20160420114245543](/images/mqqueue.png)

消费者消费指定主题的消息，并注册监听器，打印消息并根据收到的消息顺序返回不同的消费状态

- 实现**MessageListenerOrderly**的consumeMessage方法
- CONSUME_FROM_LAST_OFFSET：第一次启动从队列最后位置消费，后续再启动接着上次消费的进度开始消费 
- CONSUME_FROM_FIRST_OFFSET：第一次启动从队列初始位置消费，后续再启动接着上次消费的进度开始消费 
- CONSUME_FROM_TIMESTAMP：第一次启动从指定时间点位置消费，后续再启动接着上次消费的进度开始消费 

```
    public static void main (String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_group_name");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        consumer.subscribe("TopicTest", "TagA || TagC || TagD");

        consumer.registerMessageListener(new MessageListenerOrderly() {

            AtomicLong consumeTimes = new AtomicLong(0);
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs,
                                                       ConsumeOrderlyContext context) {
                context.setAutoCommit(false);
                System.out.printf(Thread.currentThread().getName() + " Receive New Messages: " + msgs + "%n");
                this.consumeTimes.incrementAndGet();
                if ((this.consumeTimes.get() % 2) == 0) {
                    return ConsumeOrderlyStatus.SUCCESS;
                } else if ((this.consumeTimes.get() % 3) == 0) {
                    return ConsumeOrderlyStatus.ROLLBACK;
                } else if ((this.consumeTimes.get() % 4) == 0) {
                    return ConsumeOrderlyStatus.COMMIT;
                } else if ((this.consumeTimes.get() % 5) == 0) {
                    context.setSuspendCurrentQueueTimeMillis(3000);
                    return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                }
                return ConsumeOrderlyStatus.SUCCESS;

            }
        });

        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
```

#### 广播模式
- 消费者设置：
        //set to broadcast mode
        consumer.setMessageModel(MessageModel.BROADCASTING);
        
#### 定时消息
1. 生产者

```
          // This message will be delivered to consumer 10 seconds later.
             message.setDelayTimeLevel(3);
```

2. 消费者
消费者监听器中可以取得消息的存储时间，进而可以计算出延迟的时间

```
 for (MessageExt message : messages) {
                     // Print approximate delay time period
                     System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                             + (System.currentTimeMillis() - message.getStoreTimestamp()) + "ms later");
                 }
```

#### 批量处理
- 能够提高小的消息的传送效率
- 限制1：same topic, same waitStoreMsgOK and no schedule support.
- 限制2：the total size of the messages in one batch should be no more than 1MiB.

```
String topic = "BatchTest";
List<Message> messages = new ArrayList<>();
messages.add(new Message(topic, "TagA", "OrderID001", "Hello world 0".getBytes()));
messages.add(new Message(topic, "TagA", "OrderID002", "Hello world 1".getBytes()));
messages.add(new Message(topic, "TagA", "OrderID003", "Hello world 2".getBytes()));
try {
    producer.send(messages);
} catch (Exception e) {
    e.printStackTrace();
    //handle the error
}
```
- 太大则需要分块生产

```
public class ListSplitter implements Iterator<List<Message>> {
    private final int SIZE_LIMIT = 1000 * 1000;
    private final List<Message> messages;
    private int currIndex;
    public ListSplitter(List<Message> messages) {
            this.messages = messages;
    }
    @Override public boolean hasNext() {
        return currIndex < messages.size();
    }
    @Override public List<Message> next() {
        int nextIndex = currIndex;
        int totalSize = 0;
        for (; nextIndex < messages.size(); nextIndex++) {
            Message message = messages.get(nextIndex);
            int tmpSize = message.getTopic().length() + message.getBody().length;
            Map<String, String> properties = message.getProperties();
            for (Map.Entry<String, String> entry : properties.entrySet()) {
                tmpSize += entry.getKey().length() + entry.getValue().length();
            }
            tmpSize = tmpSize + 20; //for log overhead
            if (tmpSize > SIZE_LIMIT) {
                //it is unexpected that single message exceeds the SIZE_LIMIT
                //here just let it go, otherwise it will block the splitting process这里直接让这个单独的“大”消息过去，否则会阻塞拆分的进程
                if (nextIndex - currIndex == 0) {
                   //if the next sublist has no element, add this one and then break, otherwise just break如果下个子列表没有元素，则把这个“大”消息加进去，并把nextIndex+1
                   nextIndex++;  
                }
                break;
            }
            if (tmpSize + totalSize > SIZE_LIMIT) {
                break;
            } else {
                totalSize += tmpSize;
            }
    
        }
        List<Message> subList = messages.subList(currIndex, nextIndex);
        currIndex = nextIndex;
        return subList;
    }
}
//then you could split the large list into small ones:
ListSplitter splitter = new ListSplitter(messages);
while (splitter.hasNext()) {
   try {
       List<Message>  listItem = splitter.next();
       producer.send(listItem);
   } catch (Exception e) {
       e.printStackTrace();
       //handle the error
   }
}
```

#### 消息过滤选择
- 一条消息只能有一个tag，因此通过consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");这种方式的选择性不强
- 可以通过向message中添加1到n条属性，消费者对属性或属性组合进行逻辑判断来过滤消息

```
// 生产者Set some properties.
msg.putUserProperty("a", String.valueOf(i));
//消费者
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");
```
#### OpenMessaging国际标准
[OpenMessaging](https://openmessaging.github.io/)

[新闻](https://www.csdn.net/article/a/2017-10-18/15933580)

#### Logappender日志
#### Transaction事务处理


