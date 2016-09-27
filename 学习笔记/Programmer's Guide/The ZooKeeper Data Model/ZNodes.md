# The ZooKeeper Data Model - ZooKeeper的数据模型
<p>ZooKeeper 具有像分布式文件系统的分层的命名空间，唯一的区别是命名空间中的每个节点都
可以有与当前节点及其子节点相关的数据。类似于文件系统，允许包含一个文件或目录。节点的路径
只能规范的表述为“/”开头的绝对路径，不存在相对引用。unicode 字符都可以设置在路径中，除
了以下限制：</p>

- 空字符\u0000不能作为路径的一部分，会引起 C binding 问题；
- \u0001 - \u001F 和 \u007F  - \u009F 的字符不要使用，because they don't display well, or render in confusing ways；
- \ud800 - uF8FF, \uFFF0 - uFFFF 不能使用；
- "."字符可以作为名称的一部分，但是"."和".."不能单独使用表明一个路径，因为zookeeper不支持相对哦路径。例如："/a/b/./c" or "/a/b/../c" 均不合法；
- zookeeper 为保留字段；

## ZNodes - 节点
<p>ZooKeeper 树中的每个节点代表一个 znode，每个 znode 保存了一系列状态数据包括节点数据
更改和 acl 更改的版本信息（version）。version信息结合时间戳 timestamp 可以确保 ZooKeeper 
验证节点缓存实现一致性的更新操作。当节点数据发生改变，其版本信息 version 就会加一。客户端获取
的节点信息也包含对应的 version 信息。当客户端需要对节点数据进行更新或删除操作，客户端需要提供
操作该节点的 version 信息，如果客户端提供的 version 信息和节点数据的 version 不匹配，将
无法实现更新或删除操作。</p>

**注:** 在分布式应用领域，node 代表一个通用的服务器、组成集群的一员、客户端的操作等。在ZooKeeper文档中一个 znode 表示一个数据节点；
Servers 代表构成 ZooKeeper 服务的服务器；quorum peers 代表组成ZooKeeper集群的服务器；client 代表所有的请求 ZooKeeper 服务的请求方。

## Watches - 监听

<p>ZooKeeper 通常以远程服务的方式访问，客户端每次访问 znode 时都需要获取 znode 的内容，这样的代价会非常大，会造成
更高的延迟。ZooKeeper 采用基于通知（notification）的机制：卡勒乎段向 ZooKeeper 注册需要接受通知的znode，通过对
znode 设置监视点 watch来接受通知。监视点是一个单词触发的操作，意即 znode 发生改变会触发客户端的监视，ZooKeeper 会
向客户端发送通知，然后会清除该监视点，监视点只会触发一个通知。为了接受多个通知，客户端必须每次通知后设置一个新的监视点。</p>

## Data Access - 数据访问
<p>存储在 znode 中的数据的读写操作都是原子性的，Read 操作读取 znode 所有的字节数据，Write 操作将覆盖所有的数据。
每个节点都包含一个访问控制列表Access Control List (ACL)以限制节点可访问的权限。<br/>
ZooKeeper 用于管理数据一致性，包括配置信息、状态信息等。各种需要协调一致性的数据余个共同点：数据量比较小，kb量级。ZooKeeper
客户端和服务器端通过相关机制检查以确保节点的数据小于1M，并且这些数据的平均值要小于 1M。操作相对较大的数据会造成其他操作的延迟，
约为较大的数据需要花费更多的时间进行网络传输和媒体存储。如果需要存储大数据量，通常的处理方式是采用大型的存储系统例如NFS、HDFS等，
而将存储的位置信息存储在 ZooKeeper 中。</p>

## Ephemeral Nodes - 临时节点
<p>客户端请求ZooKeeper创建会话，ZooKeeper 会创建一个临时节点（Ephemeral Nodes），客户端提交给
ZooKeeper 的所有操作均关联在该会话之上，当会话结束时，临时节点也将删除。由于这个机制，临时节点
不允许包含子节点。</p>

## Sequence Nodes - 序列节点
<p>当创建一个节点时可以请求 ZooKeeper 在该节点的路径后 append 一个单调递增的数字（计数器）。这个计数器相对于父节点时唯一的。
计数器的格式是 %010d：表示数字10, 0 padding。计数器这种表示方式为了简化排序。例如一个节点的路径表示为"<path>0000000001"。
<b>注：</b>计数器用于存储父节点将要引用的下一个序号。计数器为4字节的int数据，当超过最大值时会溢出。</p>

# ZooKeeper Stat Structure
ZooKeeper 的 znode 节点的stat结构有以下字段构成：
- czxid：创建该 znode 节点的 change id
- mzxid：改变该 znode 节点的 change id
- pzxid：最后一次改变该节点下的孩子节点 change id
- ctime：创建该 znode 节点时的毫秒时间
- mtime：改变该 znode 节点时的毫秒时间
- version：znode 数据发生更改的版本信息
- cversion：znode 的子节点发生改变的版本信息
- aversion：znode 的 ACL(Access Control List) 发生改变的版本信息
- ephemeralOwner：对于临时节点，与之相关联的 session id
- dataLength：znode 节点中数据的长度
- numChildren：该 znode 节点的孩子节点数目