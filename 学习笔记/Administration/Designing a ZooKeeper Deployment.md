# Designing a ZooKeeper Deployment
ZooKeeper的可靠性取决于以下两个基本假设：

- 实际部署的服务器集群中只有少数服务器会出现错误，如服务器崩溃或网络出现故障。
- 部署的机器能够正常运行。包括正确的执行程序、包含一定的存储、正常的网络通信。

下面讨论的是从 Single-Machine 和 Cross-Machine 两个角度入手，如何部署 ZooKeeper 最大程度确保两条假设的正确性。

## Single Machine Requirements
<p>当集群中有超过半数的服务器之间能够正常相互通信，则整个 ZooKeeper 集群的才能对外提供服务。假设系统需要容错的服务器数是 F，
则需要部署 2F+1 台服务器（奇数台服务器构成）。<br/>
未达到最大限度的容错率，可尝试采取措施使出错的服务器相互独立，比如：如果多台机器共享同一台电源、冷却系统等，会造成一定相关性的错误。
</p>

## Cross Machine Requirements
<p>如果 ZooKeeper 需要和其他的应用竞争 CPU 资源、网络资源、存储资源等时，其性能会明显收到影响。
ZooKeeper 需要这些资源的较为长久的保障，这意味着它使用存储媒体来记录更改之前，负责更改的操作需要被完成。
所以需要确保 ZooKeeper 的操作没有因为媒体资源的竞争而终止或挂起。以下操作可降低这些竞争造成的影响：</p>

- ZooKeeper 的事务日志记录要在一个专用的磁盘中进行（一个专门的分区是不够的）。ZooKeeper 顺序进行日志记录，不需要搜索是否存在与之相竞争的进程。
- Do not put ZooKeeper in a situation that can cause a swap. In order for ZooKeeper to function with any sort of timeliness, it simply cannot be allowed to swap. Therefore, make certain that **the maximum heap size given to ZooKeeper is not bigger than the amount of real memory available to ZooKeeper**. For more on this, see [Things to Avoid](http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_commonProblems) below.

