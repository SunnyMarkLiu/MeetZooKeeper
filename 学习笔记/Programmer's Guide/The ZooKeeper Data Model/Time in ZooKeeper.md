# Time in ZooKeeper - ZooKeeper 中的时间
1. Zxid
<p>ZooKeeper 的状态改变都会接受到以 zxid(ZooKeeper Transaction Id) 形式的时间戳，
包含了 ZooKeeper 改变的所有顺序信息，每次改变对应唯一的 zxid。例如：如果 zxid1 小于 zxid2,
表明，zxid1 在 zxid2 之前发生。</p>

2. Version numbers
每个节点发生改变，都会时节点的版本 version 信息增加。其中包含三种 version：

- version：znode 数据发生更改的版本信息
- cversion：znode 的子节点发生改变的版本信息
- aversion：znode 的 ACL(Access Control List) 发生改变的版本信息

3. Ticks
<p>当 ZooKeeper 集群使用多服务器时，服务器使用心跳时间 tick 定义事件的发生：状态的upload、
session超时、连接超时等。session 超时的最小时间是两倍的心跳时间 tick time。</p>

4. Real time
ZooKeeper 不使用实际的时间或时钟时间，除了在创建 znode 和修改 znode 时添加的时间戳。
