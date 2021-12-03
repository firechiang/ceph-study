#### 一、创建或修改Ceph Metadata Server(MDS)集群（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 创建或调整文件系统元数据存储服务节点
# cephfs 是元数据存储服务集群名称（如果是创建的话名字可以随便起）
# --placement 指定集群节点数量
$ ceph orch apply mds cephfs --placement="3"

# 在server001节点上添加一台MDS节点，属于集群名cephfs
# 注意：不推荐使用该命令创建MDS节点，建议使用第一条命令创建MDS
#$ ceph orch daemon add mds cephfs server001
```

#### 二、删除Ceph Metadata Server(MDS)集群
```bash
# 先标记名字叫cephfs的Ceph Metadata Server(MDS)集群为禁用状态
# 注意：MDS服务集群名称在WEB控制界面Cluster > Services栏可以查看
$ ceph mds fail mds.cephfs

# 再删除名字叫cephfs的Ceph Metadata Server(MDS)集群服务
$ ceph orch rm mds.cephfs
```