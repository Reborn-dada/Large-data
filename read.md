虚拟机：hadoop 123456
虚拟机中mysql：/usr/local/mysql/bin  ./mysql -uroot -p123456
启动mysql服务：service mysql start

Hadoop-2.6.0-cdh5.7.0
hive-1.1.0-cdh5.7.0

为什么很多公司选择使用Hadoop作为大数据平台的解决方案
1)源码开源
2)社区活跃、参与者众多
3)设计分布式存储和计算方方面面
	Flume进行数据采集、
	Spark/MapReduce/Hive等进行数据处理、
	HDFS/Hbase进行数据存储
4)已得到企业界的验证

注意：不同的问题场景选择不同的框架来解决就好了。

================================================================================================================================================
https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html
HDFS的设计目标
非常巨大的分布式文件系统
运行在普通廉价的硬件上
易扩展、为用户提供性能不错的文件存储服务等等

1 Master  带N Slaves
HDFS/YARN/Hbase
An HDFS architecture = master->NameNode + slaves->DataNodes
1个文件会被拆分成多个Block
blocksize:128M
130M==>2个Block：128M和2M
分布式存储：会把一个文件分成多个block，存储在各个节点（一台电脑一个节点）

NameNode
1)负责客户端请求的响应
2)负责元数据（文件的名称、副本系数、Block存放的DN）的管理。客户端发起读请求时，首先要去NN上查一下，要读的文件存在哪里，因为已经把文件分成多个block，分散的存在各个机器上。 

DataNode
1）存储用户的文件对应的数据块（block）
2）要定期向NN发送心跳信息，汇报本身及其所有的block信息，健康状况

client
就是你的操作。例如hdfs上shell操作，java api文件

A typical deployment has a dedicated machine that runs only the NameNode software. Each of the other machines in the cluster runs one instance of the DataNode software. The architecture does not preclude running multiple DataNodes on the same machine but in a real deployment that is rarely the case.
== NameNode + N个DataNode
NameNode是主进程。管理的东西比较多所以自己一个进程，其他机器每个机器部署一个DataNode，建议：NN和DN是部署在不同的节点上。

HDFS副本机制
replication factor 副本系数、副本因子
All blocks in a file except the last block are the same size
================================================================================================================================================

/home/hadoop
	software:存放的是安装的软件包
	app:存放的是所有软件的安装目录
	data：存放的是所有使用的测试数据目录
	source：存放的是软件源码目录，eg：spark

hadoop环境搭建
1)下载Hadoop
https://archive.cloudera.com/cdh5/cdh/5/
hadoop-2.6.0-cdh5.7.0
wget https://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0/

2)安装jdk
	下载
	解压到app目录：tar -zxvf jdk-7u51-linux-x64.tar.gz -C /home/hadoop/hadoop/app/
	验证安装时候成功：~/app/jdk1.7.0_51/bin  ./java -version
	建议把bin目录配置到系统环境变量(~/.bash_profile)中
		export JAVA_HOME=/home/hadoop/hadoop/app/jdk1.7.0_51
		export PATH=$JAVA_HOME/bin:$PATH

3)机器参数设置
	hostname:hadoop001

	修改机器名：vi /etc/sysconfig/network
		NETWORKING=yes
		HOSTNAME=hadoop001

	设置ip和hostname的映射关系：vi /etc/hosts
		10.45.34.1 hadoop001

	ssh免密码登录(本步骤可以不做，在重启hadoop时候就需要手动输入密码)
		ssh-keygen -t rsa
		cp id_rsa.pub authorized_keys (id_rsa.pub 在hadoop目录下隐藏了)

4)hadoop配置文件修改
	/home/hadoop/hadoop/app/hadoop-2.6.0-cdh5.7.0/etc/hadoop下
	hadoop-env.sh
		export JAVA_HOME=/home/hadoop/hadoop/app/jdk1.7.0_51

	core-site.xml
		<!--指定namenode的地址-->
		<property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop001:8020</value>
   		</property>
   		<!--用来指定使用hadoop时产生文件的存放目录-->
   		<property>
            <name>hadoop.tmp.dir</name>
            <value>/home/hadoop/hadoop/app/tmp</value> 
   		</property>

   	hdfs-site.xml
   		<!--指定hdfs保存数据的副本数量-->
   		<property>
            <name>dfs.replication</name>
            <value>1</value>
    	</property>

5)格式化HDFS
	注意：这一步操作，只是在第一次执行，每次如果都格式化的话，hdfs上的数据就都没有啦
	/home/hadoop/hadoop/app/hadoop-2.6.0-cdh5.7.0/bin下
	./hdfs namenode -format

6)启动HDFS
	sbin下 ./start-dfs.sh

	验证是否启动成功
	jps
		2580 NameNode
		2983 Jps
		2680 DataNode
		2874 SecondaryNameNode
	浏览器
		http://hadoop001:50070

7)停止HDFS
	sbin下 ./stop-dfs.sh

注意：hadoop下文档目录
		bin 存放的是hadoop、yarn、hdfs客户端的脚本
		etc 配置文件相关的
		sbin 存放的是hadoop集群、yarn、hdfs服务端的脚本（启停它们时候用）
================================================================================================================================================
HDFS shell常用命令的使用
	/bin
	hadoop fs 
	hadoop fs -ls /
	hadoop fs -mkir /a/b
	hadoop fs -rm -r /a
	hadoop fs -put xxx /a      	将本地文件上传到hdfs上
	hadoop fs -get xxx xxxxx	将hdfs上文件下载到本地重命名为xxxxx

================================================================================================================================================
HDFS优点
	高容错
	适合批处理
	适合大数据处理
	可构建在廉价的机器上
HDFS缺点
	低延迟的数据访问（导致不能快速查询，所以需要借助HBase，它就相当于大数据的数据库，只要你的主键设计很好的话，查询超级快的）
	小文件存储不合适

================================================================================================================================================
MapReduce概述
（工作场景少了，只做了解，主要还是Spark）
MapReduce适合的场景
离线批处理
MapReduce不适合的场景
实时计算
流式计算
DAG计算

MapReduce编程模式
1.input
2.map&reduce
3.output

input => splitting  => Mapping          =>             shuffing           =>        reducing           =>  output
文件     进行拆分操作   拆分出每一部分都用一个map做处理   把相同key的融合进一个reduce中   根据需求计算(求和)		结果

================================================================================================================================================
YARN架构
1 RM(ResourceManager) + N NM(NodeManager)
ResourceManager的职责：一个集群active状态的RM只有一个，负责整个集群的资源管理和作业调度
1)处理客户端的请求(启动/杀死作业)
2)启动/监控ApplicationMaster(一个作业对应一个AM， 对应一个应用程序),一旦AM挂了，在其他节点上就会再启一个AM
3)监控NM(通过心跳，如果说这个NodeManager挂了，这个NodeManager上的任务就要告诉对应的ApplicationMaster进行处理)
4)系统的资源分配和调度

NodeManager：整个集群中有N个，它负责单个节点的资源管理和使用以及task的运行情况
1)定期向RM汇报本节点的资源使用情况和各个Contariner的运行状态
2)接受并处理RM的Container启停的各种命令
3)单个节点的资源管理和任务管理

ApplicationMaster：每个应用/作用对应一个，负责应用程序管理
1)数据切分
2)为应用程序向RM申请资源(Container)，并分配内部任务
3)与NM通信以启停task，task是运行再containre中的
4)task的监控和容错

container：
对任务运行情况的描述(cpu,内存RAM,磁盘等)

YARN执行流程
1)首先客户端向yarn提交作业
2)ResourceManager为该作业分配第一个Container（这个container是去运行ApplicationMaster的）
3)RM会与对应的NM通信，要求NM在这个container上启动应用程序的AM
4)AM首先向RM注册，然后AM将为各个任务申请资源，并监控运行情况
5)AM采用轮询的方式通过RPC协议向RM上申请和领取资源
6)AM申请到资源后，便和相应的NM通信，要求NM启动任务
7)NM启动我们作业对应的task

yarn环境搭建
mapred-site.xml
	<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
yarn-site.xml
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

启动yarn：sbin/start-yarn.sh
验证是否启动成功
	jps
		2156 ResourceManager
		2247 NodeManager
	web：http://hadoop001:8088

提交MapReduce作业到yarn运行
hadoop jar /home/hadoop/hadoop/app/hadoop-2.6.0-cdh5.7.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.7.0.jar wordcount /input/wc/hello.text /output/wc
当我们再次执行该作业时，会报错，需要删除该目录再运行
FileAlreadyExistsException: Output directory hdfs://hadoop001:8020/output/wc already exists

================================================================================================================================================
大数据数据仓库工具Hive

Hive产生背景
	MapReduce变成的不便性
	HDFS上的文件缺少Schema（hdfs上文件就是一个文件，没有类似于关系型数据库中表名、列名、列数据类型等。）

Hive底层的执行引擎有很多MapReduce、Tez、Spark...
压缩：GZIP、LZO、Snappy、BZIP2..
存储：TextFile、SequenceFile、RCFile、ORC、Parquet
UDF：自定义函数

为什么要使用Hive
	简单容易上手（提供类似SQL查询语言HQL）
	为超大数据集设计的计算/存储扩展能力（MR计算，HDFS存储）（MR计算不够的时可横向扩展NodeManager）
	统一的元数据管理（可与Presto/Impala/SparkSQL等共享数据）

Hive环境搭建
1)Hive下载：https://archive.cloudera.com/cdh5/cdh/5/
	wget https://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.7.0.tar.gz/

2)解压
	tar -zxvf hive-1.1.0-cdh5.7.0.tar.gz -C /home/hadoop/hadoop/app/

3)配置
	系统环境变量(~/.bash_profile)
		export HIVE_HOME=/home/hadoop/hadoop/app/hive-1.1.0-cdh5.7.0
		export PATH=$HIVE_HOME/bin:$PATH

	事先安装一个mysql

	hive-env.sh
		HADOOP_HOME=/home/hadoop/hadoop/app/hadoop-2.6.0-cdh5.7.0

  	hive-site.xml
	  	<?xml version="1.0"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

		<configuration>
		<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://localhost:3306/sparksql?createDatabaseIfNotExist=true</value>
		</property>
		 
		<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
		</property>
		 
		<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
		</property>
		 
		<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>123456</value>
		</property>
		</configuration>

4)拷贝mysql驱动包到/home/hadoop/hadoop/app/hive-1.1.0-cdh5.7.0/lib

5)启动Hive
	/home/hadoop/hadoop/app/hive-1.1.0-cdh5.7.0/bin ./hive
	要注意hadoop和mysql是否启动，若没有启动，在启动hive时会失败

使用Hive完成wordcount
create table hive_wordcount (context string);

将txt数据导入hive表中
load data local inpath '/home/hadoop/hadoop/data/hello.txt' into table hive_wordcount;

select word,count(1) from hive_wordcount lateral view explode(split(context,'\t')) wc as word group by word;

lateral view explode() 是把每行记录按照指定分割符进行拆解

hive ql 提交执行以后会生成mapreduce作业，并在yarn上运行

案例：员工与部门
create table emp (
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int
)ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

create table dept(
deptno int,
dname string,
location string
)ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

load data local inpath '/home/hadoop/hadoop/data/emp.txt' into table emp;
load data local inpath '/home/hadoop/hadoop/data/dept.txt' into table dept;


================================================================================================================================================
Spark产生背景
由于
mapreduce局限性
1)代码繁琐
2)只能够支持map和reduce方法
3)执行效率低下；
4)不适合迭代多次、交互式、流式处理

框架多样化
1)批处理（离线处理）：MapReduce、Hive、Pig
2)流式处理（实时处理）：Storm
3)交互式计算：Impala
所以
Spark诞生

================================================================================================================================================
Spark 2.1.0源码编译

前置工作
1)Building Spark using Maven requires Maven 3.3.9 and Java 7.
2)export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m -XX:MaxPermSize=512M"
./dev/change-scala-version.sh 2.11
 
mvn编译命令：
./build/mvn -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver -Dhadoop.version=2.6.0-cdh5.7.0 -DskipTests clean package


./dev/make-distribution.sh --name 2.6.0-cdh5.7.0 --tgz -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver -Dhadoop.version=2.6.0-cdh5.7.0

坑1：Failed to execute goal on project spark-yarn_2.11: Could not resolve dependencies for project org.apache.spark:spark-yarn_2.11:jar:2.1.0: Failed to collect dependencies at org.apache.hadoop:hadoop-yarn-server-web-proxy:jar:2.6.0-cdh5.7.0: Failed to read artifact descriptor for org.apache.hadoop:hadoop-yarn-server-web-proxy:jar:2.6.0-cdh5.7.0: Could not transfer artifact org.apache.hadoop:hadoop-yarn-server-web-proxy:pom:2.6.0-cdh5.7.0 from/to cloudera (https://repository.cloudera.com/artifactory/cloudera-repos): Remote host closed connection during handshake: SSL peer shut down incorrectly -> [Help 1]
相似的有很多找不到lancher.jar 等等等

在pom.xml指定位置添加：cloudera 和 aliyun 的库，一般都能解决。
    <repository>
      <id>cloudera</id>
      <name>cloudera Repository</name>
      <url>https://repository.cloudera.com/artifactory/cloudera-repos</url>
   	</repository>
   	

================================================================================================================================================
Spark环境搭建

local模式
解压到app
bin下rm -rf *.cmd
配置环境变量vim .bash_profile
bin下spark-shell --master local[2]


standalone
conf/spark-env.sh
SPARK_MASTER_HOST=hadoop001
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
SPARK_WORKER_INSTANCES=1
sbin下./start-all.sh启动

eg：我有10台机器
想要这样
hadoop1:master
hadoop2:worker
hadoop3:worker
hadoop4:worker
...
hadoop10:worker

则需要在slaves配置文件中添加
hadoop2
hadoop3
hadoop4
...
hadoop10

这样./start-all.sh 会在hadoop1机器上启动master进程，在slaves文件配置的所有hostname的机器上启动worker

spark wordcount统计
val file =spark.sparkContext.textFile("file:///home/hadoop/hadoop/data/wc.txt")
val wordCounts=file.flatMap(line=>line.split(",")).map((word=>(word,1))).reduceByKey(_+_)
wordCounts.collect

================================================================================================================================================
Spark SQL概述

一、spark前世今生

Hive：类似于sql的Hive QL语言
	  sql => mapreduce 所以效率不好
	  改进：hive on tez、hive on spark、hive on mapreduce

Spark：hive on Spark => shark
	  shark推出：受欢迎，基于spark、基于内存的列式存储、与hive能够兼容
	  缺点：hive ql的解析、逻辑执行计划生成、执行计划的优化是依赖于hive的
	  ，shark仅仅是把hive的物理执行计划从mr作业替换成spark作业。

Shark终止后，产生了2个分支
1）hive on spark
	Hive社区，源码在Hive中
2）Spark SQL
	Spark社区，源码在Spark中
	支持多种数据源，多种优化技术，扩展性好很多


二、SQL on Hadoop
1)Hive 
	sql => mapreduce
	metastore:元数据 
	sql：database、table、view
	facebook

2)impala
	cloudera:cdh(建议大家在生产上使用的hadoop系列版本)、cm
	sql：自己的守护进程执行的，非mapreduce
	metastore

3)presto
	facebook
	sql

4)drill
	sql
	直接访问：hdfs、rdbms、json、hbase、mangodb、s3、hive

5)Spark SQL
	sql
	datafram/dataset api编程
	metastore
	直接访问：hdfs、rdbms、json、hbase、mangodb、s3、hive =>外部数据源

三、Spark SQL概述
	Spark SQL is Apache Spark's module for working with structured data.
	处理结构化数据

	1.Spark SQL允许您使用SQL或熟悉的DataFrame API在Spark程序中查询结构化数据。可用于Java，Scala，Python和R
	2.DataFrames和SQL提供了访问各种数据源的常用方法，包括Hive，Avro，Parquet，ORC，JSON和JDBC。您甚至可以跨这些来源加入数据。
	3.Spark SQL支持HiveQL语法以及Hive SerDes和UDF，允许您访问现有的Hive仓库
	4.通过JDBC或ODBC连接。
	5.性能和可扩展性
	6.社区
	详见官网https://spark.apache.org/sql

	1)spark Sql的应用并不局限于sql
	2)访问hive、json、parquet等文件的数据
	3)sql只是Spark SQL的一个功能而已=>spark sql这个名字并不恰当
	4)Spark SQL提供了SQL的api、DataFrame和Dataset的api

四、Spark SQL愿景
	write less code
	read less data
	let the optimizer do the hard work优化工作交给底层，不用我们做

五、Spark SQL架构
	hive AST (sql语法树)=>
	spark program       =>没有解析完整的逻辑执行计划  => 逻辑执行计划=> 优化后的逻辑执行计划 =>物理执行计划 =>物理执行计划优化 =>生产运行
	streaming sql    	=>                        +
										元数据信息schema catalog					 

================================================================================================================================================
从Hive平滑的过渡到spark
SQLContext/HiveContext/SparkSession的使用(前两个1.x，后面是2.x)
spark-shell/spark-sql这两个命令行的使用
thriftserver/beeline的使用
jdbc方式访问编程

一、SQLContext的使用
	Spark1.x中Spark SQL的入口点：SQLContext
	val sc: SparkContext
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

提交spark application到环境中运行
./bin/spark-submit \
  --class SparkSql.SQLContextApp \
  --master local[2] \
  /home/hadoop/lib/sql_jar.jar \
  /home/hadoop/hadoop/data/test.json

  ./bin/spark-submit --class SparkSql.SQLContextApp --master local[2] /home/hadoop/lib/sql_jar.jar /home/hadoop/hadoop/data/test.json

Exception in thread "main" java.lang.SecurityException: Invalid signature file digest for Manifest main attributes
报这个错原因：因为依赖jar包中的META-INF中有多余的.SF文件与当前jar包冲突
zip -d sql.jar 'META-INF/.SF' 'META-INF/.RSA' 'META-INF/*SF'


./spark-submit --class SparkSql.HiveContextApp --master local[2] --jars /home/hadoop/hadoop/app/hive-1.1.0-cdh5.7.0/lib/mysql-connector-java-5.1.46.jar /home/hadoop/lib/sql_hive.jar










