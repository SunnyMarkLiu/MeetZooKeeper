# ZooKeeper Sessions
<p>当使用 c/java binding 创建一个 handler 请求服务时，ZooKeeper 客户端建立与 ZooKeeper 
通信的 session 会话。会话一旦创建，handler 状态为 CONNECTING，客户端视图向 ZooKeeper 
服务器请求建立服务，建立连接服务后状态切换为 CONNECTED。正常的状态处于 CONNECTING 和 CONNECTED。
如果发生不可自修复的错误，比如 session 过期，验证失败，如果应用过期关闭 handler，handler 的状态
就会切换到 CLOSED 状态。ZooKeeper 客户端的状态切换如下图所示：</p>
