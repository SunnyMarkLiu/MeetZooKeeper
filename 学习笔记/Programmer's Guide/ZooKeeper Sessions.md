# ZooKeeper Sessions
<p>当使用 c/java binding 创建一个 handler 请求服务时，ZooKeeper 客户端建立与 ZooKeeper 
通信的 session 会话。会话一旦创建，handler 状态为 CONNECTING，客户端视图向 ZooKeeper 
服务器请求建立服务，建立连接服务后状态切换为 CONNECTED。正常的状态处于 CONNECTING 和 CONNECTED。
如果发生不可自修复的错误，比如 session 过期，验证失败，如果应用过期关闭 handler，handler 的状态
就会切换到 CLOSED 状态。ZooKeeper 客户端的状态切换如下图所示：</p>

![handler state](https://github.com/SunnyMarkLiu/MeetZooKeeper/blob/master/%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/handler_state.jpg)

<p>客户端为了创建与服务器端的 session 会话，程序中需要提供连接的字符串数据，包括：host:port对，
每个 host:port 代表连接到相应 ZooKeeper 服务器，例如："127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"。
ZooKeeper 客户端从中取出一个地址并尝试连接；如果连接失败，或者客户端由于其他原因断开连接，客户端会自动取出下一个服务器地址，
并尝试连接，直到该连接实现建立（或重新建立）。</p>

<p>当客户端获得一个连接 ZooKeeper 服务的 handler，ZooKeeper 服务器创建一个会话 session 分配给客户端，session会话代表一个 64 位的数字。
当客户端连接到其他的 ZooKeeper 服务器，客户端会在握手期间（handshake）携带 session id。考虑安全机制，服务器端会为该 session id 创建 password，
其他的 ZooKeeper 服务器可以对连接进行校验。密码会在创建 session 期间由服务器通过 session id 发送给客户端。客户端就会通过携带 password 的 session id 
给其他服务器以建立新的连接。</p>

<p>ZooKeeper 客户端创建 session 的参数 timeout，客户端发送请求超时，服务器端响应超时。当前版本要求的超时时间最小不小于 tick time 的2倍（tick 时间在zoo.cfg
文件中配置），最大值不超过20倍的 tick time。</p>

<p>当客户端（session）成为 ZooKeeper 服务集群的一个分区时，客户端开始搜索在 session 会话创建期间指定的服务器列表。最终，当客户端与服务器列表中的一个服务器的连接活动
重新连接时，session 的状态切换为“connected”（重新建立的时间要小于超时时间 timeout）,或者切换为“expired”状态（如果重新建立连接的时间超过 timeout ）。在断开状态，
不建议创建 session 对象（ZooKeeper.class）。ZooKeeper 客户端会自动为你创建 handler 进行重连接（客户端使用相关启发式算法）。只有在 session 过期时才会需要你手动
创建 session 对象。</p>

<p>ZooKeeper 集群管理会话 session 的超时，而不是客户端。当客户端与 ZooKeeper 建立 session 会话，客户端会提供 timeout 参数值，服务器端
使用 timeout 决定客户端会话是否超时。当集群没有在超时时间 timeout 内监听到客户端时，服务器集群即将客户端标记为超时，并且集群会删除所有与该 session 
相关的临时节点（ephemeral node）。客户端将一直保持断开状态直到 TCP 重新与集群建立连接，对于会话超时，watcher 监视会接受到 “session expired”通知。</p>

<p>ZooKeeper 会话创建调用的另一个参数是默认的监视器（default watcher）。当客户端的状态发生改变时，都会通知 watcher 监视器。例如：如果客户端与服务器端断开连接，
客户端将接受到通知。在新的连接建立时，watcher 监听器接收到到的第一个事件时 session connection 会话连接事件。</p>

<p>客户端持续请求 ZooKeeper 服务器，session 会话将一直有效。如果 session 空闲了一段时间，会导致会话超时，客户端可通过发送 PING 请求与服务器保持连接使 session 会话保持有效。
PING 命令不仅可以确保让服务器端直到客户端连接时有效的，还可以让客户端直到所连接的 ZooKeeper 服务器是有效的。PING 指令的时间足够确保客户端能够检测除连接是否断开并重新建立连接。</p>

<p>一旦客户端与服务器端的连接成功建立（session state 为 connected），有以下两种基本的情况会导致客户端生成 connectionloss：</p>

1. 客户端应用调用一个非存活的 session时；
2. ZooKeeper 客户端与服务器断开连接，但此时任存在一个对服务器挂起的操作时；

## Updating the list of servers-服务器列表更新
<p>ZooKeeper 允许客户端通过提供的host:port对更新连接的字符串（所连接的集群的ip和port对），每个 ip:port 代表一个 
ZooKeeper 服务器。客户端会调用概率  负载均衡算法，会导致客户端断开当前与服务器的连接，从而达到新的服务器列表中的每个服务
所连接的数目均衡，即达到负载均衡的效果。</p>
