#### 一、集群组件相关介绍
 - Ceph-Deploy：快速部署 Ceph 集群的工具（注意：该组件已不再维护）
 - CephFS: Ceph文件系统提供了一个符合posix标准的文件系统，它使用Ceph存储集群在文件系统上存储用户数据。与RBD和RGW一样，CephFS服务也作为librados的本机接口实现
 - Ceph Monitors(MON)：集群健康协调服务，Ceph监视器通过保存集群状态的映射来跟踪整个集群的健康状况(注意：该服务组件最好是基数个节点，因为它类似于Zookeeper的paxos算法)
 - Ceph Manager: Ceph Manager守护进程(Ceph -mgr)是在Kraken版本中引入的，它与monitor守护进程一起运行，为外部监视和管理系统提供额外的监视和接口
 - Ceph Metadata Server(MDS): 跟踪文件层次结构，仅为Ceph FS文件系统存储元数据
 - Ceph Rados(RBD): 负责存储对象操作，不管对象的数据类型如何。Rados层确保数据始终保持一致。为此，它执行数据复制、故障检测和恢复，以及跨集群节点的数据迁移和再平衡
 - Librados: librados库是一种访问Rados的方便方法，支持PHP、Ruby、Java、Python、C和c++编程语言。它为Ceph存储集群(Rados)提供了本机接口，并为其他服务提供了基础，如RBD、RGW和CephFS，这些服务构建在librados之上。librados还支持从应用程序直接访问Rados，没有HTTP开销
 - Rados Gateway(RGW)：提供对象存储服务。它使用librgw (Rados网关库)和librados，允许应用程序与Ceph对象存储建立连接。RGW提供了与Amazon S3和OpenStack Swift兼容的RESTful api接口
 - Object Storage Device(OSD)： 对象存储组件，数据以对象的形式存储在OSD中。一个OSD守护进程绑定到集群中的一个物理磁盘。通常来说，Ceph集群中物理磁盘的总数与在每个物理磁盘上存储用户数据的OSD守护进程的总数相同
 - RBD：提供持久块存储，它是瘦配置的、可调整大小的，并在多个OSD上存储数据条带。RBD服务被构建为一个在librados之上的本机接口
 
#### 二、[使用Cephadm搭建集群（推荐生产使用）][1] 
#### 三、[手动搭建集群（已废弃）][2]
#### 四、[创建修改存储池以及快照相关（集群使用必须有存储池）][3]
#### 五、[创建修改删除Ceph Metadata Server(MDS)（文件系统元数据存储服务集群）][4]
#### 六、[创建修改删除CephFs文件系统（创建文件系统必须有Ceph Metadata Server(MDS)集群）][5]
#### 七、[客户端用户授权管理（生成客户端用户使用Ceph集群所需要的Token）][6]
#### 八、[Ceph客户端以及文件系统简单使用][7]
#### 九、[集群服务管理相关（停止，启动，重启相关服务）][8]
#### 十、[WEB管理界面出现警告或错误相关处理包括禁用警告或错误提示][9]

![image](https://github.com/firechiang/ceph-study/blob/master/image/ceph-framework.jpg)

[1]: https://github.com/firechiang/ceph-study/tree/master/docs/setup-cluster-cephadm.md
[2]: https://github.com/firechiang/ceph-study/tree/master/docs/setup-cluster-node.md
[3]: https://github.com/firechiang/ceph-study/tree/master/docs/edit-pool.md
[4]: https://github.com/firechiang/ceph-study/tree/master/docs/metat-server.md
[5]: https://github.com/firechiang/ceph-study/tree/master/docs/edit-cephfs.md
[6]: https://github.com/firechiang/ceph-study/tree/master/docs/client-auth.md
[7]: https://github.com/firechiang/ceph-study/tree/master/docs/client-simple-use.md
[8]: https://github.com/firechiang/ceph-study/tree/master/docs/service-manager.md
[9]: https://github.com/firechiang/ceph-study/tree/master/docs/waring-handler.md
