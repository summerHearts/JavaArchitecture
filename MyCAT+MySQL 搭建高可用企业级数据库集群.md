##MyCAT+MySQL 搭建高可用企业级数据库集群
 
 - MyCat是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是==分表分库==，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

 - MyCat发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL、SQL Server、Oracle、DB2、PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。而在最终用户看来，无论是那种存储方式，在MyCat里，都是一个传统的数据库表，支持标准的SQL语句进行数据的操作，这样一来，对前端业务系统来说，可以大幅降低开发难度，提升开发速度。

 - MyCat是一个数据库中间层。
 
 - MyCat可以实现对后端数据库的分库分表和读写分离。
 - MyCat对前端应用隐藏了后端数据库的存储逻辑。

##2、MyCAT数据库中间层的主要作用
- 1、作为分布式数据库中间层使用

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-17-44.png)
  
- 2、实现后端数据库的读写分离及负载均衡
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-21-41.png)
  
- 3、对业务数据库进行垂直切分
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-24-16.png)
  
- 4、对业务数据库进行水平切分
  
   ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-26-16.png)
   
- 5、控制数据库连接的数量
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-30-55.png)
  

##3、MyCat的基本元素
- 逻辑库
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-33-33.png)

- 逻辑表
   
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-35-10.png)

- 逻辑表的类别
 
   ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_17-36-41.png)

  

##4、MyCat安装集成

```
wget http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20180122220033-linux.tar.gz

tar zxf /Mycat-server-1.6.5-release-20180122220033-linux.tar.gz

java -version

rpm -qa | grep java

rpm -e --nodeps  tzdata-java-2018c-1.e16.noarch

mv mycat/ /usr/local/

vim /etc/profile 

source /etc/profile

mycat start
```

##5、MyCat进阶
 
 - 1、详解MyCat主要配置文件的标签和属性
   ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_19-25-46.png)
   
   为了快速跑起一个Mycat demo，我们先在本地数据库里面建立test1和test2数据库，创建一个名为opt的表，字段为id（int）及name（varchar）

    Mycat的主要配置文件在其conf目录下面，分别是server.xml、schema.xml和rule.xml，为快速启动mycat，我们按照配置的顺序和主要配置项说明。
    
    - 1、server.xml 服务配置 
       - 这里主要配置mycat的用户和权限信息，这里的账户用于后面连接mycat使用。 
       - 快速入门可以简单这样配置： 
         schemas是后面schema.xml里面配置的DB，例如：我这里配置了一个用户名为user，密码为666666的账户，默认只能访问test的DB
   
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License"); 
	- you may not use this file except in compliance with the License. - You 
	may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0 
	- - Unless required by applicable law or agreed to in writing, software - 
	distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT 
	WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the 
	License for the specific language governing permissions and - limitations 
	under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<!-- 配置该属性的时候一定要保证mycat的字符集和mysql 的字符集是一致的 -->	
	<property name="charset">utf8</property>  
    <!-- 指定每次分配socker direct buffer 的值，默认是4096字节 -->	
	<property name="processorBufferChunk">4096/property>   
    <!-- 配置系统可用的线程数量，默认值为CPU核心X每个核心运行线程的数量 -->	
	<property name="processors">4/property>   
    
    <!-- 指定BufferPool 的计算比例  默认值为bufferChunkSize(4096)X processors X 1000
	<property name="processorBufferPool">100000000/property> -->   
    
    <!-- 用来控制ThreadLocalPool 分配Pool的比例大小，默认值为100
	<property name="processorBufferLocalPercent">100/property> -->

    <!-- 用来指定Mycat全局序列类型，0为本地文件，1为数据库方式，2为时间戳列方式，默认使用本地文件方式，文件方式主要用于测试
	<property name="sequnceHandlerType">0/property> -->

    <!-- TCP 参数配置，mycat在每次建立前后端连接时候，都会使用这些参数初始化TCP属性，详细可以查看Java API 文档：http://docs.oracle.com/javase/7/docs/api/net/StandardSocketOptions.html
	<property name="frontSocketSoRcvbuf">1024*1024/property>
	<property name="frontSocketSoSndbuf">4*1024*1024/property>
	<property name="frontSocketNoDelay">1/property>
	<property name="backSocketSoRcvbuf">4*1024*1024/property>
	<property name="backSocketSoSndbuf">1024*1024/property>
	<property name="backSocketNoDelay">1/property> -->	

    <!-- mysql 连接相关配置 -->
    <!-- <property name="packetHeaderSize">4</property>  指定mysql协议中的报文头长度，默认4个字节-->
	<!-- <property name="maxPacketSize">1024*1024*16</property> 配置可以携带的数据量最大值，默认16M-->
	<!-- <property name="idleTimeout">1024*1024*16</property> 指定连接的空闲时间超时长度，如果某个连接空闲时间超过该值，则将连接关闭并回收，单位为毫秒，默认值为30分钟-->
	<!-- <property name="txIsolation">3</property> 初始化前端连接事务的隔离级别有：
		READ_UNCOMMITTED=1
		READ_COMMITTED=2
		REPEATED_READ=3
		SERIALIZABLE=4 
      默认为3-->
	<!-- <property name="sqlExecuteTimeout">3</property>执行sql超时时间，默认为300秒-->


	<!-- 心跳属性配置 -->
    <!-- <property name="processorCheckPeriod">1000</property>清理前后端空闲、超时、关闭连接的时间间隔，单位为毫秒，默认为1秒-->
    <!-- <property name="dataNodeIdleCheckPeriod">300*1000</property>对后端连接进行空闲，超时检查的时间间隔，单位为毫秒，默认为300秒-->
    <!-- <property name="dataNodeHeartbeatPeriod">10*1000</property>对后端所有读写库发起心跳的间隔时间，单位为毫秒，默认为10秒-->

	<!-- 服务相关属性 -->
    <!-- <property name="bindIp">0.0.0.0</property>mycat服务监听的ip地址，默认为0.0.0.0-->
    <!-- <property name="serverPort">8066</property>定义mycat使用的端口，默认值为8066-->
    <!-- <property name="managerPort">9066</property>定义mycat管理的端口，默认值为9066-->

	<!-- 分布式事务开关属性 -->
    <!-- <property name="handleDistributedTransactions">0</property>0为不过滤分布式事务，1过滤分布式事务，2不过滤分布式事务，但是记录分布式事务日志。主要用户是否允许跨库事务。mycat 1.6版本开始，支持此属性-->

 
    <!-- <property name="useOffHeapForMerge">1</property>配置是否启用非堆内存跨分片结果集，1为开启，0为关闭，mycat1.6开始支持该属性-->

    <!-- 全局表一致性检测 -->
	<property name="useGlobleTableCheck">0</property>  <!--通过添加_MYCAT_OP_TIME字段来进行一致性检测，为BIGINT类型 1为开启全加班一致性检测、0为关闭 -->

	<property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->


      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
      <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
	 
	<!-- 
	<property name="processors">1</property> 
	<property name="processorExecutor">32</property> 
	 -->
		<!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena-->
		<property name="processorBufferPoolType">0</property>
		<!--默认是65535 64K 用于sql解析时最大文本长度 -->
		<!--<property name="maxStringLiteralLength">65535</property>-->
		<!--<property name="processorExecutor">16</property>-->
		<!--
			<property name="serverPort">8066</property> <property name="managerPort">9066</property> 
			<property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property> 
			<property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
		<!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
		<property name="handleDistributedTransactions">0</property>
		
		<!--单位为m-->
		<property name="memoryPageSize">1m</property>

		<!--单位为k-->
		<property name="spillsFileBufferSize">1k</property>

		<property name="useStreamOutput">0</property>

		<!--单位为m-->
		<property name="systemReserveMemorySize">384m</property>

		<!--是否采用zookeeper协调切换  -->
		<property name="useZKSwitch">true</property>

	</system>
	
	<!-- 全局SQL防火墙设置 -->
	<!-- 
	<firewall> 
	   <whitehost>
	      <host host="127.0.0.1" user="mycat"/>
	      <host host="127.0.0.2" user="mycat"/>
	   </whitehost>
       <blacklist check="false">
       </blacklist>
	</firewall>
	-->
	<!-- 定义登录mycat对的用户权限 -->
	<user name="root">
		<property name="password">123456</property>
		<!-- 若要访问TESTDB 必须现在server.xml 中定义，否则无法访问TESTDB-->
		<property name="schemas">TESTDB</property>
		<!-- 配置是否允许只读 -->
		<property name="readOnly">true</property>
		<!-- 定义限制前端整体的连接数，如果其值为0，或者不设置，则表示不限制连接数量 -->
		<property name="benchmark">11111</property>
		<!-- 设置是否开启密码加密功能，默认为0不开启加密，为1则表示开启加密 -->
		<property name="usingDecrypt">1</property>
		<!-- 表级 DML 权限设置 -->
		<!-- 		
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>		
		 -->
	</user>
</mycat:server>
```
  
   用户密码的加密
   
```
   java -cp Mycat-server-1.65-release.jar io.mycat.util.DecryptUtil 0:root:123456
   
   vim server.xml 
   
   <property name= "usingDecrypt">1</property>
   
   <property name="password">NSJSALKSLAKSAKSLAKSL==</property>
   
   mysql -uroot -p -p9066 -h127.0.0.1
   
```

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_20-07-04.png)

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-143_20-07-19.png)
        
  - 2、schema.xml 数据库配置
         - 这里主要配置数据库信息，一个schema就是一个逻辑库，可以理解为Mycat管理的一个数据库DB（实际上不存在，是一个虚拟的概念）name属性和server.xml里面对应。 
         - table是Mycat的逻辑表，这里我们配置一个名为opt的逻辑表 
         - dataNode属性是将其绑定在真实数据库中的数据节点中，这里我们配置了两个dataNode，分别是test1数据库和 test2数据库，注意，这两个数据库的真实表名需和table的name一致 
         - dataHost属性就是我们熟悉的mysql连接，这里我们都用本地作为一台服务器，也可以配置不同服务器上的不同mysql，这里还可以配置读写分离，我们先不考虑。 
         - rule属性就是分片规则，这里用的是名称，需在下一步的rule.xml里面定义好

        ```
         <schema name="test" checkSQLschema="false" sqlMaxLimit="100">
         <!-- 分片表配置 -->
                 <table name="opt" dataNode="dn$1-2" rule="rule1" />
         </schema>

         <dataNode name="dn1" dataHost="localhost1" database="test1" />
         <dataNode name="dn2" dataHost="localhost1" database="test2" />

         <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
         
         <heartbeat>select user()</heartbeat>
         
         <writeHost host="hostS1" url="localhost:3306" user="root"
               password="666666" />
               
         </dataHost>
        ```
         
- 3、rule.xml 规则配置
       - 这里我们主要配置的是分片规则，Mycat支持很多分片规则，我们使用的是一个较为简单的取模规则，即根据columns标签里面的id属性来取模，count属性定义为数据节点的个数。

        ```
        <tableRule name="rule1">
           <rule>
              <columns>id</columns>
              <algorithm>mod-long</algorithm>
           </rule>
        </tableRule>

        <function name="mod-long" class="io.mycat.route.function.PartitionByMod">
        <!-- how many data nodes -->
             <property name="count">2</property>
        </function>
        ```


 

   

