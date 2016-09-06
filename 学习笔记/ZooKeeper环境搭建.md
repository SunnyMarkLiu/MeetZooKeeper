# ZooKeeper环境搭建
## 下载最新稳定版的 Zookeeper
使用的是最新稳定版的 [zookeeper-3.4.9](http://mirror.bit.edu.cn/apache/zookeeper/stable/)

## System Requirements
![System Requirements](https://github.com/SunnyMarkLiu/MeetZooKeeper/tree/master/%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/SystemRequirements.png)

## Required Software
ZooKeeper 运行在 java 1.7 以上版本，FreeBSD 系统支持 openjdk7。

## Standalone Operation
1、 将下载的 ZooKeeper 解压，进入 conf 目录，复制一份 zoo_sample.cfg 的配置文件命名为 zoo.cfg，修改如下：

    # the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats
    tickTime=2000
    # the directory where the snapshot is stored.
    # do not use /tmp for storage, /tmp here is just 
    # example sakes.
    dataDir=/var/lib/zookeeper
    # the port at which the clients will connect
    clientPort=2181
    
2、 进入 bin 目录，启动 ZooKeeper：

    ./zkServer.sh start
    
启动运行后会创建相关目录，所以注意相关目录的写权限。启动的日志如下：

    ZooKeeper JMX enabled by default
    Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
    
3、 Client Connecting to ZooKeeper

客户端连接到 ZooKeeper：

    ./zkCli.sh -server 127.0.0.1:2181
    
连接成功的日志输出：

    Welcome to ZooKeeper!
    2016-09-05 09:53:14,485 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server 127.0.0.1/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
    JLine support is enabled
    2016-09-05 09:53:14,625 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@876] - Socket connection established to 127.0.0.1/127.0.0.1:2181, initiating session
    [zk: 127.0.0.1:2181(CONNECTING) 0] 2016-09-05 09:53:14,800 [myid:] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 127.0.0.1/127.0.0.1:2181, sessionid = 0x156f80a3ea00000, negotiated timeout = 30000
    
    WATCHER::
    
    WatchedEvent state:SyncConnected type:None path:null
    
    [zk: 127.0.0.1:2181(CONNECTED) 0] 

4、 基本指令操作

（1）help 帮助指令，列出可操作的相关指令

    ZooKeeper -server host:port cmd args
    	stat path [watch]
    	set path data [version]
    	ls path [watch]
    	delquota [-n|-b] path
    	ls2 path [watch]
    	setAcl path acl
    	setquota -n|-b val path
    	history 
    	redo cmdno
    	printwatches on|off
    	delete path [version]
    	sync path
    	listquota path
    	rmr path
    	get path [watch]
    	create [-s] [-e] path data acl
    	addauth scheme auth
    	quit 
    	getAcl path
    	close 
    	connect host:port

（2）create /test_znode test_data

    [zk: 127.0.0.1:2181(CONNECTED) 2] create /test_znode test_data
    Created /test_znode
    
（3）ls /

    [zk: 127.0.0.1:2181(CONNECTED) 3] ls /
    [zookeeper, test_znode]
    
（4）get /test_znode

    [zk: 127.0.0.1:2181(CONNECTED) 4] get /test_znode
    test_data
    cZxid = 0x6
    ctime = Mon Sep 05 09:59:46 CST 2016
    mZxid = 0x6
    mtime = Mon Sep 05 09:59:46 CST 2016
    pZxid = 0x6
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 9
    numChildren = 0
    
（5）set /test_znode other_data

    [zk: 127.0.0.1:2181(CONNECTED) 5] set /test_znode other_data
    cZxid = 0x6
    ctime = Mon Sep 05 09:59:46 CST 2016
    mZxid = 0x7
    mtime = Mon Sep 05 10:02:20 CST 2016
    pZxid = 0x6
    cversion = 0
    dataVersion = 1
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 10
    numChildren = 0

（6）delete /test_znode

    [zk: 127.0.0.1:2181(CONNECTED) 6] delete /test_znode
    [zk: 127.0.0.1:2181(CONNECTED) 7] ls /
    [zookeeper]
    
## Clustered (Multi-Server) Setup

<p>为获取稳定的服务，需要将 ZooKeeper 部署在分布式集群中，由于 ZooKeeper 通过仲裁机制
检测系统是否可以正常对外提供服务，所以集群选择奇数个服务器进行搭建。构建容错集群最小的服务器数是3台服务器。</p>

1、 每台服务器中，同 Standalone Operation 一样，在 zookeeper 解压目录下的 conf 目录中创建 zoo.cfg 配置文件：

    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/var/lib/zookeeper
    # the port to listen for client connections; that is, the port that clients attempt to connect to.
    clientPort=2181
    server.1=192.168.1.111:2888:3888
    # Ubuntu_Server_64_1
    server.2=192.168.1.133:2888:3888
    # Ubuntu_Server_64_2
    server.3=192.168.1.135:2888:3888

其中，`server.id=host:port:port`的配置说明：

- server.id：在 zoo.cfg 文件中配置的配置的 dataDir 路径下（即：/var/lib/zookeeper）创建`myid`文件，里面写入 `id` 值
- host：构建 zookeeper 集群的服务器的 ip 地址
- port：第一个 port 用于连接 leader 服务器；第二个 port 用于 leader election

2、 每台服务器中，在 zoo.cfg 文件中配置的配置的 dataDir 路径下（即：/var/lib/zookeeper）创建`myid`文件
注意 myid 文件中写入该服务器的 id 标识，注意只能有一行，不能包括其他多余的字符；并且 该 id 要与 zoo.cfg 中配置的 server.id 相对应（ip要对应）

3、 启动 zookeeper 集群
（1）启动其中一个、zookeeper 服务器

    ./zkServer.sh start
    
（2）尝试连接服务器

    telnet 192.168.1.111 2181
    
输出信息后输入 `stat`：

    markliu@Linux:/usr/local/zookeeper-3.4.9/bin$ telnet 192.168.1.135 2181
    Trying 192.168.1.135...
    Connected to 192.168.1.135.
    Escape character is '^]'.
    stat
    This ZooKeeper instance is not currently serving requests
    Connection closed by foreign host.
    
由于只启动了一个服务器，所以整个 zookeeper 集群不能对外提供服务，zookeeper 集群中正常运行的服务器需要多余未正常运行的服务器。

（2）启动另外一个 zookeeper 服务器，并 telnet 连接服务器

    markliu@Linux:/usr/local/zookeeper-3.4.9/bin$ telnet 192.168.1.135 2181
    Trying 192.168.1.135...
    Connected to 192.168.1.135.
    Escape character is '^]'.
    stat
    Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
    Clients:
     /192.168.1.111:43624[0](queued=0,recved=1,sent=0)
    
    Latency min/avg/max: 0/0/0
    Received: 1
    Sent: 0
    Connections: 1
    Outstanding: 0
    Zxid: 0x200000000
    Mode: leader
    Node count: 4
    Connection closed by foreign host.

（3）客户端连接到 ZooKeeper：

    ./zkCli.sh -server 192.168.1.135:2181

输出日志：

    markliu@Linux:/usr/local/zookeeper-3.4.9/bin$ ./zkCli.sh -server 192.168.1.135:2181Connecting to 192.168.1.135:2181
    2016-09-05 21:30:22,677 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.9-1757313, built on 08/23/2016 06:50 GMT
    2016-09-05 21:30:22,727 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=localhost
    2016-09-05 21:30:22,727 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_91
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/local/jdk1.8.0_91/jre
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/usr/local/zookeeper-3.4.9/bin/../build/classes:/usr/local/zookeeper-3.4.9/bin/../build/lib/*.jar:/usr/local/zookeeper-3.4.9/bin/../lib/slf4j-log4j12-1.6.1.jar:/usr/local/zookeeper-3.4.9/bin/../lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper-3.4.9/bin/../lib/netty-3.10.5.Final.jar:/usr/local/zookeeper-3.4.9/bin/../lib/log4j-1.2.16.jar:/usr/local/zookeeper-3.4.9/bin/../lib/jline-0.9.94.jar:/usr/local/zookeeper-3.4.9/bin/../zookeeper-3.4.9.jar:/usr/local/zookeeper-3.4.9/bin/../src/java/lib/*.jar:/usr/local/zookeeper-3.4.9/bin/../conf::.:/usr/local/jdk1.8.0_91/lib:/usr/local/jdk1.8.0_91/jre/lib
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
    2016-09-05 21:30:22,741 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=3.16.0-30-generic
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=markliu
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/markliu
    2016-09-05 21:30:22,742 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/usr/local/zookeeper-3.4.9/bin
    2016-09-05 21:30:22,744 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=192.168.1.135:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@506c589e
    Welcome to ZooKeeper!
    2016-09-05 21:30:23,097 [myid:] - INFO  [main-SendThread(192.168.1.135:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server 192.168.1.135/192.168.1.135:2181. Will not attempt to authenticate using SASL (unknown error)
    JLine support is enabled
    2016-09-05 21:30:23,578 [myid:] - INFO  [main-SendThread(192.168.1.135:2181):ClientCnxn$SendThread@876] - Socket connection established to 192.168.1.135/192.168.1.135:2181, initiating session
    2016-09-05 21:30:23,600 [myid:] - INFO  [main-SendThread(192.168.1.135:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 192.168.1.135/192.168.1.135:2181, sessionid = 0x356fa8189d40002, negotiated timeout = 30000
    
    WATCHER::
    
    WatchedEvent state:SyncConnected type:None path:null
    [zk: 192.168.1.135:2181(CONNECTED) 0] 
