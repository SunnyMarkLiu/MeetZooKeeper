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

    When a client gets a handle to the ZooKeeper service, ZooKeeper creates a ZooKeeper session, 
    represented as a 64-bit number, that it assigns to the client. If the client connects to a 
    different ZooKeeper server, it will send the session id as a part of the connection handshake. 
    As a security measure, the server creates a password for the session id that any ZooKeeper 
    server can validate.The password is sent to the client with the session id when the client 
    establishes the session. The client sends this password with the session id whenever 
    it reestablishes the session with a new server.