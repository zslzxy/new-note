# Kafka

## 1. kafka操作

### 1. 1 Kafka集群部署

> - 第一步：解压文件：
>
> ```
>  tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/ 
> ```
>
> - 第二步：修改解压缩文件名：
>
> ```
> mv kafka_2.11-0.11.0.0/ kafka
> ```
>
> - 第三步：进入kafka目录，创建logs文件夹；
>
> ```
>  mkdir logs 
> ```
>
> - 第四步：进入config目录，修改server.properties文件
>
> ```
> #broker 的全局唯一编号，不能重复 
> broker.id=0 
> #删除 topic 功能使能 
> delete.topic.enable=true 
> #处理网络请求的线程数量 
> num.network.threads=3 
> #用来处理磁盘 IO 的现成数量 
> num.io.threads=8 
> #发送套接字的缓冲区大小 
> socket.send.buffer.bytes=102400 
> #接收套接字的缓冲区大小 
> socket.receive.buffer.bytes=102400 
> #请求套接字的缓冲区大小 
> socket.request.max.bytes=104857600 
> #kafka 运行日志存放的路径 
> log.dirs=/opt/module/kafka/logs 
> #topic 在当前 broker 上的分区个数 
> num.partitions=1 
> #用来恢复和清理 data 下数据的线程数量 
> num.recovery.threads.per.data.dir=1 
> #segment 文件保留的最长时间，超时将被删除 
> log.retention.hours=168 
> #配置连接 Zookeeper 集群地址 
> zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181
> ```
>
> - 第五步：配置环境变量,并source：
>
> ```
> #KAFKA_HOME 
> export KAFKA_HOME=/opt/module/kafka 
> export PATH=$PATH:$KAFKA_HOME/bin 
> ```
>
> - 第六步：分发kafka,并修改server.properties文件中的broker.id；
>
> ```
>  xsync kafka/ 
> ```

### 1.2 kafka启动、关闭

> 启动集群，分别在所有机器中输入启动命令：
>
> ```
> bin/kafka-server-start.sh -daemon config/server.properties
> ```
>
> 关闭集群，分别在所有机器中输入命令：
>
> ```
> bin/kafka-server-stop.sh stop 
> ```



