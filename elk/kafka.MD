## kafka

1. 下载

2. 启动
2.1 启动zookeeper
``` bash
nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties > zk.out &
```

2.2 启动Kafka
``` bash
nohup ./bin/kafka-server-start.sh config/server.properties > kfk.out &
```

3. 测试和使用
3.1 新建一个话题Topic
Topic的名字叫"test",只有一个分区和一个备份。
``` bash
 bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
查看存在的Topic列表:
``` bash
 bin/kafka-topics.sh --list --zookeeper localhost:2181
```

4. 发送消息
Kafka提供了一个命令行的工具，可以从输入文件或者命令行中读取消息并发送给Kafka集群。每一行是一条消息。
``` bash

bin/kafka-console-producer.sh --broker-list localhost:9092  --topic test 
bin/kafka-console-producer.sh --broker-list 0.0.0.0:9092 --topic test  
This is a message
This is another message

```

5. 消费消息
Kafka也提供了一个消费消息的命令行工具。

``` bash
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
bin/kafka-console-consumer.sh --bootstrap-server localhost:2181 --topic test --from-beginning
```


./kafka-topics.sh --create --zookeeper 192.168.0.61:2181,192.168.0.62:2181 --replication-factor 2 --partitions 1 --topic shequ

./kafka-topics.sh --list --zookeeper 192.168.0.61:2181,192.168.0.62:2181
'''在一台服务器上创建一个发布者'''#创建一个broker，发布者
./kafka-console-producer.sh --broker-list 192.168.0.61:9092 --topic shequ
 


 ./bin/kafka-console-consumer.sh --zookeeper 192.168.0.61:2181,192.168.0.62:2181  --topic shequ --from-beginning
./bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.61:2181,192.168.0.62:2181 --topic shequ --from-beginning

./bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.61:2181,192.168.0.62:2181 --topic shequ --from-beginning --partition 0

### kafka manager
1. 下载打包
https://github.com/yahoo/kafka-manager
https://github.com/yahoo/kafka-manager/releases

2. 修改配置并运行
在解压后的conf目录中打开 application.conf文件，修改其中的配置信息，最主要的内容为：
``` bash
kafka-manager.zkhosts="localhost:2181"
```
配置为Kafka的 zookeeper 服务器。你还可以通过环境变量ZK_HOSTS配置这个参数值。

其他详细的配置信息参考官方文档：https://github.com/yahoo/kafka-manager

#### 启动
在解压的目录中，使用下面的命令启动Kafka-manager。
``` bash
> bin/kafka-manager
```
默认情况下端口为9000，你还可以通过下面的命令指定配置文件和端口：
``` bash
> bin/kafka-manager -Dconfig.file=/path/to/application.conf -Dhttp.port=8080

bin/kafka-manager -Dconfig.file=/path/to/application.conf -Dhttp.port=8082
```






















如果没有配置Java的环境变量，还可以通过下面的方式指定Java8的目录： 

3. 
/data/db/mongodb-linux-x86_64-3.0.6
