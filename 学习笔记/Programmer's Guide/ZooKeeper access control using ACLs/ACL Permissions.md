# ZooKeeper access control using ACLs —— ZooKeeper 使用 ACLs 实现访问控制
Zookeeper使用ACLs控制访问它的znodes(Zookeeper的数据节点)。ACL实现非常类似于UNIX文件访问权限：它使用权限位允许/不允许一个简单和范围的各种操作。
和标准的UNIX权限不同的是，一个 ZooKeeper 数据节名，不由三个标准范围(用户，用户所在用户组、其他用户组)限制。Zookeeper 没有 znode 所有者的概念。
而是一个ACLs指定一组ids和与这些ids相关联的权限。 
还要注意ACL只适用于特定的znode。尤其不适用于children。例如，如果/app对ip:192.168.1.56是只读的并且/app/status是全都可读的，任何人可以读取
/app/status；ACLs不是递归控制的。 
Zookeeper支持可插拔的权限认证方案。使用scheme:id的形式指定Ids,scheme是id对应的权限认证scheme。例如，ip:172.16.16.1是一个地址为172.16.16.1的id. 

## ACL Permissions —— ACL权限 
 Zookeeper支持下面的permissions：

- CREATE：创建子节点
- READ  ：从节点获取数据并列出它的子节点
- WRITE ：向节点设置数据
- DELETE：删除一个子节点
- ADMIN ：管理员权限，用于设置权限

CREATE和DELELTE权限已经更细粒度的划分了WRITE权限。
