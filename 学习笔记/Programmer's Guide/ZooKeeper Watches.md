# ZooKeeper Watches
ZooKeeper 的读操作 `getData(), getChildren(), exists()` - 都有设置watch的选项。ZooKeeper 中
`Watches` 的定义为：一个watch事件是 one-time 触发，向客户端发送设置 watch，当设置 watch 的数据发生变化时。
Watch 的定义中有三个关键点：

- **One-time trigger（一次触发）** 当数据发生变化时将向客户端发送一个 watch 事件。例如，
如果一个客户端用`getData("/znode1",true)`并且过一会之后 /znode1 的数据改变或删除了，
客户端将获得一个 /znode1 的watch事件。如果/znode1再次改变，将不会发送 watch 事件除非设置了新 watch。

- **Sent to the client(发送给客户端)** 这意味着事件发往客户端，但是可能在成功之前没到客户端。
Watches 是异步发往 watchers。Zookeeper提供一个顺序保证：在看到 watch 事件之前绝不会看到变化。
网络延迟或其他因素可能引起客户端看到 watches 并在不同时间返回 code。关键点是不同客户端看到的是一致性的顺序。

- **The data for which the watch was set（为数据设置watch）** 一个节点可以有不同方式改变。它帮助
Zookeeper维护两个watches：**data watches 和 child watches**。`getData()` 和 `exists()` 设置data watches。
`getChildren()` 设置 child watches。两者任选其一，它可以帮助 watches 根据类型返回。**getData()和exists()返回关于节点数据的信息**，
然而**getChildren()返回children列表**。因此，setData()将会触发znode设置的 data watch。一个成功的create()将会触发一个 data watch 
和一个父节点的 child watch。一个成功的delete()将触发一个data watch和一个child watch。

Watches是在client连接到Zookeeper服务端的本地维护。这可让 watches 轻量级的设置、保存、分发。当一个client连接到新server，
watch将会触发任何session事件。断开连接后不能接收到。当客户端重连，如果需要，先前注册的watches将会被重新注册并触发。所有的这些操作，对程序员都是透明的，
有一种例外的情况，当客户端与服务器断开链接时执行了创建或删除操作，对于 znode 的是否存在的 watch 将会丢失。

## Semantics of Watches
我们可以通过三种读取 ZooKeeper 状态的方法（exists, getData 和 getChildren）设置 watch，下面是 watch 可以触发的事件详细列表：

- Created event：Enabled with a call to exists
- Deleted event：Enabled with a call to exists, getData, and getChildren
- Changed event：Enabled with a call to exists and getData.
- Child event  ：Enabled with a call to getChildren.

## ZooKeeper 如何确保 watches 事件的触发
关于watches，Zookeeper 保证一下三点：

- Watches 和其他事件、异步恢复都是有序的，ZooKeeper 客户端保证这些事件是有序分发；
- 客户端将先看到 watche 事件，然后看到数据发生改变；
- 从 ZooKeeper 读取的 watch 事件的顺序由ZooKeeper 的 service 设置 watch 已经由客户端接受到的顺序决定的。

## 关于 Watches 需要记住的几点
- Watches是一次触发，如果你获得了一个 watch 事件，如果需要在之后再次接收到，需要重新设置另一个 watch；
- 因为 watches 是一次触发且在获得事件和发送请求得到 wathes 之间有延迟，你不能可靠的看到发生在 Zookeeper 节点的每一个变化。准备好处理这种延迟问题。
- 一个watch对象，或function/context对，对于指定的通知只能触发一次。例如，如果相同的文件通过exists和getData注册了相同的watch对象并且文件稍后删除了，watch将只会触发文件的删除通知。
- 从服务端断开连接时(比如服务器故障)，将不会得到任何watches直到重新建立连接。因为这个原因session事件被发送到所有watch处理器。使用session事件进入安全模式：断开连接时不接收事件，所以在这个模式里你的程序应该采取保守。
