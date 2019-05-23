1. 添加hosts信息


IP | Name
---|---
123.206.175.47 | rocketmq-nameserver01
123.206.175.47 | rocketmq-master01
182.254.210.72 | rocketmq-nameserver02
182.254.210.72 | rocketmq-master02

- 编辑主机名,master可有可可无但nameserver必须配置

```
[root@VM_1_209_centos java]# vim /etc/hosts
```

```
[root@VM_201_98_centos java]# vim /etc/hosts

127.0.0.1  localhost  localhost.localdomain  VM_201_98_centos

123.206.175.47 rocketmq-nameserver01
182.254.210.72 rocketmq-nameserver02


123.206.175.47 eggyer-node001
182.254.210.72 eggyer-node002

```

2. 上传解压
    1. 安装java(不再赘述)
    2. 安装rocketmq
- 解压
    
```
[root@VM_1_209_centos java]# cd /usr/local/software/
[root@VM_1_209_centos software]# tar -zxvf alibaba-rocketmq-3.2.6.tar.gz -C /usr/local/

```
- 重命名,增加版本标识

```
[root@VM_1_209_centos local]# mv alibaba-rocketmq/ alibaba-rocketmq-3.2.6
```
- 建立软连接
```
[root@VM_1_209_centos local]# ln -s alibaba-rocketmq-3.2.6 rocketmq

```
- 创建存储路径(rocketmq数据的存储位置),此处在软连接下简历store
    -
```
[root@VM_1_209_centos local]# mkdir /usr/local/rocketmq/store
[root@VM_1_209_centos local]# mkdir /usr/local/rocketmq/store/commitlog
[root@VM_1_209_centos local]# mkdir /usr/local/rocketmq/store/consumequeue
[root@VM_1_209_centos local]# mkdir /usr/local/rocketmq/store/index
[root@VM_1_209_centos local]# cd alibaba-rocketmq-3.2.6/
[root@VM_1_209_centos alibaba-rocketmq-3.2.6]# ls
LICENSE.txt  benchmark  bin  conf  issues  lib  store  test  wiki

```
- 修改rocketmq配置文件(双master模式)
    - 进入conf目录下后会发现2m-2s-async,2m-2s-sync,2m-noslave等模式,我们需要需要配置2m-noslave文件
```
[root@VM_1_209_centos local]# cd rocketmq/
[root@VM_1_209_centos rocketmq]# cd conf/
[root@VM_1_209_centos conf]# ll
total 32
drwxr-xr-x 2 52583 users 4096 Mar 28  2015 2m-2s-async
drwxr-xr-x 2 52583 users 4096 Mar 28  2015 2m-2s-sync
drwxr-xr-x 2 52583 users 4096 Mar 28  2015 2m-noslave
-rw-r--r-- 1 52583 users 7786 Mar 28  2015 logback_broker.xml
-rw-r--r-- 1 52583 users 2331 Mar 28  2015 logback_filtersrv.xml
-rw-r--r-- 1 52583 users 2313 Mar 28  2015 logback_namesrv.xml
-rw-r--r-- 1 52583 users 2435 Mar 28  2015 logback_tools.xml
[root@VM_1_209_centos conf]# 

```

- 进入2m-noslave文件
    - 这两个配置文件就代表了两个主节点的一些信息,因为有两个master主节点，所以主节点1启动依赖broker-a.properties，主节点2启动依赖broker-b.properties，如果是三个Master，那么还会有一个broker-c.properties，以此类推。
```
[root@VM_1_209_centos conf]# cd 2m-noslave/
[root@VM_1_209_centos 2m-noslave]# ls
broker-a.properties  broker-b.properties

```

- 修改配置文件broker-a.properties,broker-b.properties
    - Linux系统的编码问题
        - 执行locale命令查看系统语言
        - 设置系统环境变量LANG为en_US.UTF-8:export LANG=en_US.UTF-8或者编辑文件：vim /etc/sysconfig/i18n
```
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master， >0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver01:9876;rocketmq-nameserver02:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 0点
deleteWhen=00
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
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/storecommitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
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

```
- 修改日志配置文件
    - 进行日志文件的替换，sed是linux的替换命令
```
[root@VM_1_209_centos ~]# mkdir -p /usr/local/rocketmq/logs
[root@VM_1_209_centos ~]# cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' *.xml
[root@VM_1_209_centos conf]# 

```

- 修改JVM启动参数
    - 应该先启动nameserver集群,再启动broker集群
    - jvm启动大小
    - 进入rocketmq的bin文件夹
```
[root@VM_1_209_centos rocketmq]# cd bin
[root@VM_1_209_centos bin]# ls
mqadmin      mqbroker.exe        mqbroker.numanode3  mqfiltersrv.xml  mqshutdown  runbroker.sh
mqadmin.exe  mqbroker.numanode0  mqbroker.xml        mqnamesrv        os.sh       runserver.sh
mqadmin.xml  mqbroker.numanode1  mqfiltersrv         mqnamesrv.exe    play.sh     startfsrv.sh
mqbroker     mqbroker.numanode2  mqfiltersrv.exe     mqnamesrv.xml    README.md   tools.sh

```
- 打开runbroker会发现一些JVM配置参数
    - 调整-Xms4g -Xmx4g -Xmn2g

```
#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:PermSize=128m -XX:MaxPermSize=320m"
JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${HOME}/rmq_bk_gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

```

- 启动nameserver,使用mqnamesrv脚本(nohup sh mqnamesrv &)
    - 修改java路径
```
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/opt/taobao/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"
改为
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/java/jdk1.8.0_121
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/opt/taobao/java
#[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"


```

- 启动broker

```
[root@VM_1_209_centos bin]# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties  > /dev/null 2>&1 &
[2] 21821
[root@VM_1_209_centos bin]# jps
21828 BrokerStartup
20734 NamesrvStartup
21871 Jps
[root@VM_1_209_centos bin]# 
[root@VM_1_209_centos bin]# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-b.properties  > /dev/null 2>&1 &
[2] 21821
```

## 坑与问题

启动nameserver出现:ERROR: Please set the JAVA_HOME variable in your environment, We need Java(x64)! !!

```
打开启动脚本runserver.sh以及runbroker.sh文件，发现有如下三行：

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/opt/taobao/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

```
- 其中第二行是阿里巴巴集团内部服务器上的java目录，将这一行注释掉。然后第一行的JAVA_HOME的值改为自己机器的Java安装目录。然后再次起送mqnameserver以及mqbroker，观察日志发现启动成功.