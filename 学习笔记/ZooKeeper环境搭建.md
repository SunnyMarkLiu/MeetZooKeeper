# ZooKeeper环境搭建
## 下载最新稳定版的 Zookeeper
使用的是最新稳定版的 [zookeeper-3.4.9](http://mirror.bit.edu.cn/apache/zookeeper/stable/)

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
    