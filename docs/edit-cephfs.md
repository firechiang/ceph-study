#### 一、创建CephFs文件系统（注意：创建文件系统之前Ceph集群里面必须存在Ceph Metadata Server(MDS)（文件系统元数据存储服务集群））（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 查看Ceph集群当中所有的CephFs文件系统以及相关信息
$ ceph fs ls

# 创建cephfs文件系统
注意：文件系统的名字可以随便指定，这个名称就是供客户端使用挂载的目录名称，所以这个文件系统可以创建多个
#$ ceph fs flag set enable_multiple true(创建其它文件系统（名字不叫cephfs）需要先执行这个命令)
#$ ceph fs volume create aaafs（创建一个aaafs文件系统，这个文件系统会默认创建两个存储池）
$ ceph fs new cephfs <元数据存储所在池名称> <数据存储所在池名称>

# cephfs文件系统最多使用2个Ceph Metadata Server(MDS)元数据存储节点作为主要活动节点
$ ceph fs set cephfs max_mds 2

# cephfs文件系统使用1个Ceph Metadata Server(MDS)元数据存储节点作为备用节点
$ ceph fs set cephfs standby_count_wanted 1

# 查看所有文件系统元数据存储服务情况
$ ceph mds stat
cephfs:2 {0=cephfs.server001.orbgin=up:active,1=cephfs.server002.onuuxd=up:active} 1 up:standby
2个主要活动节点状态都是active（正在工作），1 个备用节点
```

#### 二、删除CephFs文件系统
```bash
# 标记名字叫cephfs的文件系统为禁用状态
$ ceph fs fail cephfs
bbbfs marked not joinable; MDS cannot join the cluster. All MDS ranks marked failed.

# 删除名字叫cephfs的文件系统，--yes-i-really-mean-it表示强制删除（注意：文件系统上的数据也将被删除）
ceph fs rm cephfs --yes-i-really-mean-it
```