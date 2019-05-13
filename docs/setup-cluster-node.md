#### 一、集群组件介绍以及节点分布
```bash
Ceph-Deploy：快速部署 Ceph 集群的工具。

CephFS: Ceph文件系统提供了一个符合posix标准的文件系统，它使用Ceph存储集群在文件系统上存储用户数据。与RBD和RGW一样，CephFS服务也作为librados的本机接口实现。

Ceph manager: Ceph manager守护进程(Ceph -mgr)是在Kraken版本中引入的，它与monitor守护进程一起运行，为外部监视和管理系统提供额外的监视和接口。

Ceph metadata server(MDS): 跟踪文件层次结构，仅为Ceph FS文件系统存储元数据。

Ceph Rados(RBD): 负责存储对象操作，不管对象的数据类型如何。Rados层确保数据始终保持一致。为此，它执行数据复制、故障检测和恢复，以及跨集群节点的数据迁移和再平衡。

Librados: librados库是一种访问Rados的方便方法，支持PHP、Ruby、Java、Python、C和c++编程语言。它为Ceph存储集群(Rados)提供了本机接口，并为其他服务提供了基础，如RBD、RGW和CephFS，这些服务构建在librados之上。librados还支持从应用程序直接访问Rados，没有HTTP开销。
                           
Rados Gateway(RGW)：提供对象存储服务。它使用librgw (Rados网关库)和librados，允许应用程序与Ceph对象存储建立连接。RGW提供了与Amazon S3和OpenStack Swift兼容的RESTful api接口。

Ceph Monitors(MON)：Ceph监视器通过保存集群状态的映射来跟踪整个集群的健康状况(集群类似于Zookeeper的paxos算法)。

Object Storage Device(OSD)： 对象存储组件，数据以对象的形式存储在OSD中。一个OSD守护进程绑定到集群中的一个物理磁盘。通常来说，Ceph集群中物理磁盘的总数与在每个物理磁盘上存储用户数据的OSD守护进程的总数相同。

RBD：提供持久块存储，它是瘦配置的、可调整大小的，并在多个OSD上存储数据条带。RBD服务被构建为一个在librados之上的本机接口。


----------|---------------|-----------------------------|----------------------|----------------------|------------------------------|
          |  Ceph-Deploy  |  Ceph metadata server(MDS)  |  Rados Gateway(RGW)  |  Ceph Monitors(MON)  |  Object Storage Device(OSD)  |
----------|---------------|-----------------------------|----------------------|----------------------|------------------------------|            
server002 |       Y       |                             |           Y          |           Y          |               Y              |
----------|---------------|-----------------------------|----------------------|----------------------|------------------------------|
server003 |               |              Y              |           Y          |           Y          |               Y              |
----------|---------------|-----------------------------|----------------------|----------------------|------------------------------|
server004 |               |                             |           Y          |           Y          |               Y              |
----------|---------------|-----------------------------|----------------------|----------------------|------------------------------|
```

#### 二、修改[vi /etc/hosts]文件信息
```bash
192.168.78.129 server002
192.168.78.130 server003
192.168.78.131 server004

$ scp /etc/hosts root@server002:/etc/                                              # 分发到各个机器
```

#### 三、Ceph-Deploy节点免密码登陆集群所有节点（在A机器生成一对公钥私钥，将公钥拷贝到想要登录的主机）
```bash
$ ssh-keygen                                                                       # 生成私钥和公钥（如果已经有了就不需要执行了）
$ ssh-copy-id -i ~/.ssh/id_rsa.pub server001                                       # 将公钥拷贝到 server001上（这样我们就可以直接免密码登录server001了）
$ ssh server001                                                                    # 测试面密码登陆
$ exit                                                                             # 退出登录
```

#### 四、安装时间同步器和同步机器时间（集群的每台机器都要安装）
```bash
$ yum install -y ntp                                                               # 安装时间同步器
$ rpm -qa|grep ntp                                                                 # 检查是否安装成功
$ ntpdate ntp1.aliyun.com                                                          # 同步时间（每台都要同步，这里同步的是"阿里云"时间服务器）
$ systemctl restart ntpd ntpdate && systemctl enable ntpd ntpdate                  # 重启时间同步器和开机启动自动同步时间（根据实际情况看要不要开启）
$ date                                                                             # 查看本机时间
```

#### 五、创建部署用户，集群的每台机器都要创建(Ceph不建议使用root账户部署)
```bash
$ useradd ceph-admin                                                               # 创建 ceph-admin 用户
$ echo "jiang" | passwd --stdin ceph-admin                                         # 为ceph-admin 用户创建密码，密码是：jiang
$ echo "ceph-admin ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph-admin # 为ceph-admin 用户授权，并生成授权文件
$ cat /etc/sudoers.d/ceph-admin                                                    # 查看授权文件
$ chmod 0440 /etc/sudoers.d/ceph-admin                                             # 修改授权文件权限
```

#### 六、关闭 selinux，集群的每台机器都要关闭(测试环境建议永久关闭 selinux，然后再临时关闭selinux，就不需要重启机器了)
```bash
$ cat /etc/selinux/config                                                          # 查看selinux信息
$ sestatus                                                                         # 查看selinux状态
$ sed -i "/^SELINUX/s/enforcing/disabled/" /etc/selinux/config                     # 永久关闭 selinux（重启机器生效）
$ setenforce 0                                                                     # 临时关闭selinux(建议使用，安装好了集群再重启机器就开启了selinux)
```

#### 七、配置防火墙，开放Ceph必要端口，集群的每台机器都要配置(也可直接关闭防火墙)
```bash
$ sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent
$ sudo firewall-cmd --zone=public --add-service=ceph --permanent
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --zone=public --list-all
```


#### 八、Ceph-Deploy安装（我们安装的Ceph是 mimic 版本）
```bash
$ cat << EOM > /etc/yum.repos.d/ceph.repo                                          # 创建ceph.repo文件内容如下
  [ceph-noarch]
  name=Ceph noarch packages
  baseurl=https://download.ceph.com/rpm-mimic/el7/noarch
  enabled=1
  gpgcheck=1
  type=rpm-md
  gpgkey=https://download.ceph.com/keys/release.asc
  EOM                                                                              # 退出保存文件

$ sudo yum install y ceph-deploy                                                   # 安装Ceph-Deploy
```
