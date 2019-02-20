1 基本概念
1.1什么是roketmq？
RcoketMQ 是一款低延迟、高可靠、可伸缩、易于使用的消息中间件。具有以下特性：
·  支持发布/订阅（Pub/Sub）和点对点（P2P）消息模型

·  在一个队列中可靠的先进先出（FIFO）和严格的顺序传递

·  支持拉（pull）和推（push）两种消息模式

·  单一队列百万消息的堆积能力

·  支持多种消息协议，如 JMS、MQTT 等

·  分布式高可用的部署架构,满足至少一次消息传递语义

·  提供 docker 镜像用于隔离测试和云集群部署

·  提供配置、指标和监控等功能丰富的 Dashboard

1.2基本概念
1.2.1 Producer
消息生产者，生产者的作用就是将消息发送到 MQ，生产者本身既可以产生消息，如读取文本信息等。也可以对外提供接口，由外部应用来调用接口，再由生产者将收到的消息发送到 MQ。
启动流程：

Producer如何感知要发送消息的broker即brokerAddrTable中的值是怎么获得的，
1.      发送消息的时候会指定topic，如果producer集合中没有，会根据指定topic到namesrv获取topic发布信息TopicPublishInfo，并放入本地集合
2.      定时从namesrv更新topic路由信息，
Producer与broker间的心跳
Producer定时发送心跳将producer信息（其实就是procduer的group）定时发送到， brokerAddrTable集合中列出的broker上去
Producer发送消息只发送到master的broker机器，在通过broker的主从复制机制拷贝到broker的slave上去
1.2.2 Producer Group
生产者组，简单来说就是多个发送同一类消息的生产者称之为一个生产者组。在这里可以不用关心，只要知道有这么一个概念即可。
1.2.3 Consumer
消息消费者，简单来说，消费 MQ 上的消息的应用程序就是消费者，至于消息是否进行逻辑处理，还是直接存储到数据库等取决于业务需要。
1.2.4 Consumer Group
消费者组，和生产者类似，消费同一类消息的多个 consumer 实例组成一个消费者组。
1.2.5 Topic
Topic 是一种消息的逻辑分类，比如说你有订单类的消息，也有库存类的消息，那么就需要进行分类，一个是订单 Topic 存放订单相关的消息，一个是库存 Topic 存储库存相关的消息。
1.2.6 Message
Message 是消息的载体。一个 Message 必须指定 topic，相当于寄信的地址。Message 还有一个可选的 tag 设置，以便消费端可以基于 tag 进行过滤消息。也可以添加额外的键值对，例如你需要一个业务 key 来查找 broker 上的消息，方便在开发过程中诊断问题。
1.2.7 Tag
标签可以被认为是对 Topic 进一步细化。一般在相同业务模块中通过引入标签来标记不同用途的消息。
1.2.8 Broker
Broker 是 RocketMQ 系统的主要角色，其实就是前面一直说的 MQ。Broker 接收来自生产者的消息，储存以及为消费者拉取消息的请求做好准备。
1.2.9 Name Server
Name Server 为 producer 和 consumer 提供路由信息。
Namesrv名称服务，是没有状态可集群横向扩展。
1.      每个broker启动的时候会向namesrv注册
2.      Producer发送消息的时候根据topic获取路由到broker的信息
3.      Consumer根据topic到namesrv获取topic的路由到broker的信息
1.2.9.1 Namesrv功能
         接收broker的请求注册broker路由信息（包括master和slave）
         接收client的请求根据某个topic获取所有到broker的路由信息
1.2.9.2 Namesrv启动流程：

1.2.9.3 RouteInfoManager
路由信息RouteInfoManager类的管理
brokerName表示一组broker，如：一个叫brokerName=broker-a， 可能包括一个master跟它的多个slave
Map<brokerName, brokerData> brokerData 由brokerName和它的broker ids和address
id表示是master还是slave
id= 0为master  大于0为slave
Map<topic, List<queueData>>
queueData由brokerName， 读队列数，写队列数，已经权限值
Map<clusterName,Set<brokerName>>  将broker按照集群分组
Map<brokerAddr, BrokerLiveInfo> 
BrokerLiveInfo 代表一个活的broker链接由最后更新时间，一个链接channel，数据版本和Ha地址组成
Broker定时向namesrv注册并更新BrokerLiveInfo的时间戳
1.2.9.4 Namesrv与broker间的心跳
1.      Broker启动的时候会启动定时任务，每隔十秒钟会向所有namesrv发送心跳请求，同时也是注册topic信息到namesrv
2.      namesrv接收borker心跳DefaultRequestProcessor的REGISTER_BROKE事件处理，
（1）      注册broker的topic信息
（2）      构建或者更新BrokerLiveInfo的时间戳
3.      NamesrvController初始化时启动线程定时调用RouteInfoManger的scanNotActiveBroker方法来定时清理不活动的broker（默认两分钟没有向namesrv发送心跳更新BrokerLiveInfo时间戳的），比较BrokerLiveInfo的时间戳，如果过期关闭channel连接


1.3RocketMQ 架构

由这张图可以看到有四个集群，分别是 NameServer 集群、Broker 集群、Producer 集群和 Consumer 集群：
1.NameServer: 提供轻量级的服务发现和路由。 每个 NameServer 记录完整的路由信息，提供等效的读写服务，并支持快速存储扩展。
2.Broker: 通过提供轻量级的 Topic 和 Queue 机制来处理消息存储,同时支持推（push）和拉（pull）模式以及主从结构的容错机制。
3.Producer：生产者，产生消息的实例，拥有相同 Producer Group 的 Producer 组成一个集群。
4.Consumer：消费者，接收消息进行消费的实例，拥有相同 Consumer Group 的
5.Consumer 组成一个集群。
简单说明一下图中箭头含义，从 Broker 开始，Broker Master1 和 Broker Slave1 是主从结构，它们之间会进行数据同步，即 Date Sync。同时每个 Broker 与NameServer 集群中的所有节点建立长连接，定时注册 Topic 信息到所有 NameServer 中。
Producer 与 NameServer 集群中的其中一个节点（随机选择）建立长连接，定期从 NameServer 获取 Topic 路由信息，并向提供 Topic 服务的 Broker Master 建立长连接，且定时向 Broker 发送心跳。Producer 只能将消息发送到 Broker master，但是 Consumer 则不一样，它同时和提供 Topic 服务的 Master 和 Slave建立长连接，既可以从 Broker Master 订阅消息，也可以从 Broker Slave 订阅消息。

2 rocketmq安装部署
RocketMQ 集群部署模式：
1.单 master 模式
也就是只有一个 master 节点，称不上是集群，一旦这个 master 节点宕机，那么整个服务就不可用，适合个人学习使用。
2.多 master 模式
多个 master 节点组成集群，单个 master 节点宕机或者重启对应用没有影响。
优点：所有模式中性能最高
缺点：单个 master 节点宕机期间，未被消费的消息在节点恢复之前不可用，消息的实时性就受到影响。
注意：使用同步刷盘可以保证消息不丢失，同时 Topic 相对应的 queue 应该分布在集群中各个节点，而不是只在某各节点上，否则，该节点宕机会对订阅该 topic 的应用造成影响。
3.多 master 多 slave 异步复制模式
在多 master 模式的基础上，每个 master 节点都有至少一个对应的 slave。master
节点可读可写，但是 slave 只能读不能写，类似于 mysql 的主备模式。
优点： 在 master 宕机时，消费者可以从 slave 读取消息，消息的实时性不会受影响，性能几乎和多 master 一样
缺点：使用异步复制的同步方式有可能会有消息丢失的问题。
4.多 master 多 slave 同步双写模式
同多 master 多 slave 异步复制模式类似，区别在于 master 和 slave 之间的数据同步方式。
优点：同步双写的同步模式能保证数据不丢失。
缺点：发送单个消息 RT 会略长，性能相比异步复制低10%左右。
刷盘策略：同步刷盘和异步刷盘（指的是节点自身数据是同步还是异步存储）
同步方式：同步双写和异步复制（指的一组 master 和 slave 之间数据的同步）
注意：要保证数据可靠，需采用同步刷盘和同步双写的方式，但性能会较其他方式低。
2.1单机部署
1 解压alibaba-rocketmq-3.2.6.tar
tar -zxvf alibaba-rocketmq-3.2.6.tar.gz -C /usr/local/devTool/
2 配置rocketmq的环境变量，在/etc/profile最后添加
export ROCKETMQ_HOME=/usr/local/devTool/alibaba-rocketmq
export PATH=$JAVA_HOME/bin:$ROCKETMQ_HOME/bin:$PATH
使rocketmq的环境变量生效
source /etc/profile
3 给下列命令可执行权限
cd /usr/local/devTool/alibaba-rocketmq/bin/;
chmod +x mqadmin mqbroker mqfiltersrv mqshutdown  mqnamesrv
4 新建日志文件夹
cd /usr/local/devTool/alibaba-rocketmq
mkdir log
5 启动nameserver
nohup mqnamesrv 1>/usr/local/devTool/alibaba-rocketmq/log/ng.log 2>/usr/local/devTool/alibaba-rocketmq/log/ng-err.log  &
查看启动状态
$ps aux|grep java
验证nameserver是否启动
$tail -f /usr/local/devTool/alibaba-rocketmq/log/ng.log
6 启动broker，在启动borker之前需要指定nameserver地址
云主机内部访问 默认ip 远程则brokerip 要设置成外网ip
云主机有公网ip 内网ip
外网连接：
编辑broker.properties
brokerIP1=10.19.73.64

export NAMESRV_ADDR=localhost:9876
nohup sh mqbroker  -c /usr/local/devTool/alibaba-rocketmq/conf/broker.properties &
验证mqbroker是否启动
tail -f /usr/local/devTool/alibaba-rocketmq/log/mq.log

本机连接：
export NAMESRV_ADDR=localhost:9876
nohup sh mqbroker &

7 关闭nameserver broker执行的命令
mqshutdown namesrv
mqshutdown broker
2.2双master集群部署
2.2.1 基础准备
1 软硬件
两台服务器
JDK，alibaba-rocketmq-3.2.6.tar.gz，rocketmq-console.war
 
2. Hosts添加信息
192.168.100.24 rocketmq-nameserv1
192.168.100.24 rocketmq-master1
192.168.100.25 rocketmq-nameserv2
192.168.100.25 rocketmq-master2
 
ping rocketmq-nameserv1
ping rocketmq-master1
ping rocketmq-nameserv2
ping rocketmq-master2
 
2.2.2 安装配置
1.上传解压【两台机器】
cd /usr/local/devTool
# 上传alibaba-rocketmq-3.2.6.tar.gz文件至
tar -zxvf alibaba-rocketmq-3.2.6.tar.gz
mv alibaba-rocketmq alibaba-rocketmq-3.2.6
ln -s alibaba-rocketmq-3.2.6 rocketmq
 
ll /usr/local/devTool
 
2. 创建存储路径【两台机器】
cd /usr/local/devTool
mkdir /usr/local/devTool/rocketmq/store
mkdir /usr/local/devTool/rocketmq/store/commitlog
mkdir /usr/local/devTool/rocketmq/store/consumequeue
mkdir /usr/local/devTool/rocketmq/store/index
 
或者
cd /usr/local/devTool
mkdir -p rocketmq/store/{commitlog,consumequeue,index}
 
3. RocketMQ配置文件【两台机器】
vim /usr/local/devTool/rocketmq/conf/2m-noslave/broker-a.properties
vim /usr/local/devTool/rocketmq/conf/2m-noslave/broker-b.properties
 
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a|broker-b
#0 表示 Master， >0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=192.168.100.24:9876;192.168.100.25:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/devTool/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/devTool/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/devTool/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/devTool/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/devTool/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/devTool/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
 
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
----------------------------------------------------------------------- 
 
 
4. 修改日志配置文件【两台机器】
mkdir -p /usr/local/devTool/rocketmq/logs
cd /usr/local/devTool/rocketmq/conf && sed -i 's#${user.home}#/usr/local/devTool/rocketmq#g' *.xml
 
                                      #sed -i 's#${user.home}#/usr/local/devTool/rocketmq#g' *.xml
                                       
5. 修改启动脚本参数【两台机器】
vim /usr/local/devTool/rocketmq/bin/runbroker.sh            
 
JAVA_usr/devTool="${JAVA_usr/devTool} -server -Xms1g -Xmx1g -Xmn512m -XX:PermSize=128m -XX:MaxPermSize=320m"                          
                                        
                                        
vim /usr/local/devTool/rocketmq/bin/runserver.sh
JAVA_usr/devTool="${JAVA_usr/devTool} -server -Xms1g -Xmx1g -Xmn512m -XX:PermSize=128m -XX:MaxPermSize=320m"
2.2.3 启动
1. 启动 NameServer 【两台机器】
 
cd /usr/local/devTool/rocketmq/bin
nohup sh mqnamesrv &
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/namesrv.log
 
2. 启动 BrokerServer  A【192.168.100.24】
cd /usr/local/devTool/rocketmq/bin
nohup sh mqbroker -c /usr/local/devTool/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 &
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/broker.log
 
netstat -ntlp
jps
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/namesrv.log
 
 
3. 启动BrokerServer B【192.168.100.25】
cd /usr/local/devTool/rocketmq/bin
nohup sh mqbroker -c /usr/local/devTool/rocketmq/conf/2m-noslave/broker-b.properties >/dev/null 2>&1 &
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/broker.log
 
netstat -ntlp
jps
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/devTool/rocketmq/logs/rocketmqlogs/namesrv.log
 
#删除日志
cd /usr/local/devTool/rocketmq/bin
sh mqshutdown broker
cd /usr/local/devTool/rocketmq/logs/rocketmqlogs
rm -rf broker.log broker.log
2.2.4 控制台程序部署
1. RocketMQ Console
#解压unzip rocketmq-console.war
unzip rocketmq-console.war -d rocketmq-console
#修改配置
rocketmq.namesrv.addr=192.168.100.24:9876;192.168.100.25:9876
 
#启动tomcat
cd /usr/local/devTool/apache-tomcat-7.0.75/bin/
./startup.sh
2.2.5 数据清理
 
cd /usr/local/devTool/rocketmq/bin
sh mqshutdown broker
sh mqshutdown namesrv
 
 
--等待停止
rm -rf /usr/local/devTool/rocketmq/store
mkdir /usr/local/devTool/rocketmq/store
mkdir /usr/local/devTool/rocketmq/store/commitlog
mkdir /usr/local/devTool/rocketmq/store/consumequeue
mkdir /usr/local/devTool/rocketmq/store/index

或者
mkdir -p /usr/local/devTool/rocketmq/store/{commitlog,consumequeue,index}
 
--按照上面步骤重启NameServer与BrokerServer

3 java操作api
3.1  阿里巴巴（com.alibaba.rocketmq）
<dependency>
<groupId>com.alibaba.rocketmq</groupId> 	<artifactId>rocketmq-client</artifactId> 
<version>3.2.6</version> 
</dependency>
3.1.2 创建生产者 （Producer）
/**RocketMQ生产者类，在请求接口时访问该类的消息推送，以便于消费者进程的消费。
 */
public class Producer {
    public static void main(String[] args)  throws MQClientException, InterruptedException{
        //声明并初始化一个producer
        //需要一个producer group名字作为构造方法的参数，这里为producer1
        DefaultMQProducer producer = new DefaultMQProducer("producer1");

        //设置NameServer地址,此处应改为实际NameServer地址，多个地址之间用；分隔
        //NameServer的地址必须有，但是也可以通过环境变量的方式设置，不一定非得写死在代码里
        producer.setNamesrvAddr("192.168.1.129:9876;192.168.1.130:9876");
        producer.setVipChannelEnabled(false);

        //调用start()方法启动一个producer实例
        producer.start();

        //发送10条消息到Topic为TopicTest，tag为TagA，消息内容为“Hello RocketMQ”拼接上i的值
        for (int i = 0; i < 10; i++) {
            try {
                Message msg = new Message("TopicTest",// topic
                        "TagA",// tag
                  ("HelloRocketMQ"+i).getBytes(RemotingHelper.DEFAULT_CHARSET)//body
                );
                //调用producer的send()方法发送消息
                //这里调用的是同步的方式，所以会有返回结果
                SendResult sendResult = producer.send(msg);
                //打印返回结果，可以看到消息发送的状态以及一些相关信息
                System.out.println(sendResult);
            } catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }

        //发送完消息之后，调用shutdown()方法关闭producer
        producer.shutdown();

    }

}
3.1.3 创建消费者（Consumer）
public class Consumer {
    public static void main(String[] args) throws InterruptedException, MQClientException {

        //声明并初始化一个consumer
        //需要一个consumer group名字作为构造方法的参数，这里为consumer1
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer1");

        //同样也要设置NameServer地址
        consumer.setNamesrvAddr("192.168.1.129:9876;192.168.1.130:9876");
consumer.setVipChannelEnabled(false);
        //这里设置的是一个consumer的消费策略
        //CONSUME_FROM_LAST_OFFSET 默认策略，从该队列最尾开始消费，即跳过历史消息
        //CONSUME_FROM_FIRST_OFFSET 从队列最开始开始消费，即历史消息（还储存在broker的）全部消费一遍
        //CONSUME_FROM_TIMESTAMP 从某个时间点开始消费，和setConsumeTimestamp()配合使用，默认是半个小时以前
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        //设置consumer所订阅的Topic和Tag，*代表全部的Tag
        consumer.subscribe("TopicTest", "*");

        //设置一个Listener，主要进行消息的逻辑处理
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                         ConsumeConcurrentlyContext context) {
                System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);
                //返回消费状态
                //CONSUME_SUCCESS 消费成功
                //RECONSUME_LATER 消费失败，需要稍后重新消费
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        //调用start()方法启动consumer
        consumer.start();
        System.out.println("Consumer Started.");
    }

}

4 最佳实践
4.1 Producer 最佳实践
4.1.1 发送消息注意事项
1. 一个应用尽可能用一个 Topic，消息子类型用 tags 来标识，tags 可以由应用自由设置。 只有发送消息设置了tags，消费方在订阅消息时，才可以利用 tags 在 broker 做消息过滤。
message.setTags("TagA");
2. 每个消息在业务层面的唯一标识码，要设置到 keys 字段，方便将来定位消息丢失问题。 服务器会为每个消息创建索引（哈希索引），应用可以通过 topic，key 来查询这条消息内容，以及消息被谁消费。由于是哈希索引，请务必保证 key 尽可能唯一，这样可以避免潜在的哈希冲突。
// 订单 Id
String orderId = "20034568923546";
message.setKeys(orderId);
3. 消息发送成功或者失败，要打印消息日志，务必要打印 sendresult 和 key 字段。
4. send 消息方法，只要不抛异常，就代表发送成功。但是发送成功会有多个状态，在 sendResult 里定义。
 SEND_OK
消息发送成功
 FLUSH_DISK_TIMEOUT
消息发送成功，但是服务器刷盘超时，消息已经进入服务器队列，只有此时服务器宕机，消息才会丢失
 FLUSH_SLAVE_TIMEOUT
消息发送成功，但是服务器同步到 Slave 时超时，消息已经进入服务器队列，只有此时服务器宕机，消息才会丢失
 SLAVE_NOT_AVAILABLE
消息发送成功，但是此时 slave 不可用，消息已经进入服务器队列，只有此时服务器宕机，消息才会丢失
对于精确发送顺序消息的应用，由于顺序消息的局限性，可能会涉及到主备自动切换问题，所以如果sendresult 中的 status 字段不等于 SEND_OK，就应该尝试重试。对于其他应用，则没有必要这样。
5. 对于消息不可丢失应用，务必要有消息重发机制
例如如果消息发送失败，存储到数据库，能有定时程序尝试重发，或者人工触发重发。

4.1.2 消息发送失败如何处理
Producer 的 send 方法本身支持内部重试，重试逻辑如下：
1. 至多重试 3 次。
2. 如果发送失败，则轮转到下一个 Broker。
3. 这个方法的总耗时时间不超过 sendMsgTimeout 设置的值，默认 10s。
所以，如果本身向 broker 发送消息产生超时异常，就不会再做重试。
以上策略仍然不能保证消息一定发送成功，为保证消息一定成功，建议应用这样做
如果调用 send 同步方法发送失败，则尝试将消息存储到 db，由后台线程定时重试，保证消息一定到达 Broker。
上述 db 重试方式为什么没有集成到 MQ 客户端内部做，而是要求应用自己去完成，我们基于以下几点考虑
1. MQ 的客户端设计为无状态模式，方便任意的水平扩展，且对机器资源的消耗仅仅是 cpu、内存、网络。
2. 如果 MQ 客户端内部集成一个 KV 存储模块，那么数据只有同步落盘才能较可靠，而同步落盘本身性能开销较大，所以通常会采用异步落盘，又由于应用关闭过程不受 MQ 运维人员控制，可能经常会发生 kill -9 这样暴力方式关闭，造成数据没有及时落盘而丢失。
3. Producer 所在机器的可靠性较低，一般为虚拟机，不适合存储重要数据。
综上，建议重试过程交由应用来控制 

4.1.3选择 oneway 形式发送
一个 RPC 调用，通常是这样一个过程
1. 客户端发送请求到服务器
2. 服务器处理该请求
3. 服务器向客户端返回应答
所以一个 RPC 的耗时时间是上述三个步骤的总和，而某些场景要求耗时非常短，但是对可靠性要求并不高，例如
日志收集类应用，此类应用可以采用 oneway 形式调用，oneway 形式只发送请求不等待应答，而发送请求在客
户端实现层面仅仅是一个 os 系统调用的开销，即将数据写入客户端的 socket 缓冲区，此过程耗时通常在微秒级。 

4.2 Consumer 最佳实践
4.2.1消费过程要做到幂等（即消费端去重）
RocketMQ 无法避免消息重复，所以如果业务对消费重复非常敏感，务必要在业务层面去重，有以下几种去重方式
1. 将消息的唯一键，可以是 msgId，也可以是消息内容中的唯一标识字段，例如订单 Id 等，消费之前判断是否在Db 或 Tair(全局 KV 存储)中存在，如果不存在则插入，并消费，否则跳过。（实际过程要考虑原子性问题，判断是否存在可以尝试插入，如果报主键冲突，则插入失败，直接跳过）msgId 一定是全局唯一标识符，但是可能会存在同样的消息有两个不同 msgId 的情况（有多种原因），这种情况可能会使业务上重复消费，建议最好使用消息内容中的唯一标识字段去重。
2. 使用业务层面的状态机去重 

4.2.2 批量方式消费


某些业务流程如果支持批量方式消费，则可以很大程度上提高消费吞吐量，例如订单扣款类应用，一次处理一个订
单耗时 1 秒钟，一次处理 10 个订单可能也只耗时 2 秒钟，这样即可大幅度提高消费的吞吐量，通过设置 consumer
的 consumeMessageBatchMaxSize 这个参数，默认是 1，即一次只消费一条消息，例如设置为 N，那么每次消费的
消息数小于等于 N。 
4.2.3跳过非重要消息
发生消息堆积时，如果消费速度一直追不上发送速度，可以选择丢弃不重要的消息
如何判断消费发生了堆积？ 
public ConsumeConcurrentlyStatus consumeMessage(//
List<MessageExt> msgs, //
ConsumeConcurrentlyContext context) {
long offset = msgs.get(0).getQueueOffset();
String maxOffset = //
msgs.get(0).getProperty(Message.PROPERTY_MAX_OFFSET);
long diff = Long.parseLong(maxOffset) - offset;
if (diff > 100000) {
// TODO 消息堆积情况的特殊处理
return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
}
// TODO 正常消费过程
return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
}


如以上代码所示，当某个队列的消息数堆积到 100000 条以上，则尝试丢弃部分或全部消息，这样就可以快速追上
发送消息的速度。 
4.2.3 优化每条消息消费过程
举例如下，某条消息的消费过程如下
1. 根据消息从 DB 查询数据 1
2. 根据消息从 DB 查询数据 2
3. 复杂的业务计算
4. 向 DB 插入数据 3
5. 向 DB 插入数据 4
这条消息的消费过程与 DB 交互了 4 次，如果按照每次 5ms 计算，那么总共耗时 20ms，假设业务计算耗时 5ms，那么总过耗时 25ms，如果能把 4 次 DB 交互优化为 2 次，那么总耗时就可以优化到 15ms，也就是说总体性能提高了 40%。对于 Mysql 等 DB，如果部署在磁盘，那么与 DB 进行交互，如果数据没有命中 cache，每次交互的 RT 会直线上升，如果采用 SSD，则 RT 上升趋势要明显好于磁盘。个别应用可能会遇到这种情况：在线下压测消费过程中，db 表现非常好，每次 RT 都很短，但是上线运行一段时间，RT 就会变长，消费吞吐量直线下降。主要原因是线下压测时间过短，线上运行一段时间后，cache 命中率下降，那么 RT 就会增加。建议在线下压测时，要测试足够长时间，尽可能模拟线上环境，压测过程中，数据的分布也很重要，数据不同，可能 cache 的命中率也会完全不同。 

5 常见问题
1 高版本强关联 com.alibaba.fastjson
2 高版本 有vip通道
3 服务器与 消费端 版本不一致导致重启消费端 消费过消息重复消费
