#### 一、安装Ceph客户端（注意：安装Ceph客户端需要添加Ceph软件源）[Ceph软件源安装请点击](https://github.com/firechiang/ceph-study/blob/master/docs/setup-cluster-cephadm.md#41%E5%AE%89%E8%A3%85%E9%9B%86%E7%BE%A4%E5%BC%95%E5%AF%BC%E5%B7%A5%E5%85%B7cephadm%E5%92%8C%E6%B7%BB%E5%8A%A0ceph%E8%BD%AF%E4%BB%B6%E6%BA%90)
```bash
# 查看Linux内核是否大于2.6（只要内核大于2.6就默认支持块设备）
$ uname -r
# 查看系统是否支持modprobe命令
$ modprobe rbd

# 安装Ceph客户端
$ yum install ceph -y
```
#### 二、创建Ceph配置文件，就是指定集群健康协调服务的各个节点，相当于指定Zookeeper集群
```bash
# 创建Ceph集群配置文件到/etc/ceph/ceph.conf
# 可以直接在对应的节点上执行sudo ceph config generate-minimal-conf命令将输出信息拷贝过来
# 注意：server001是Ceph集群引导节点安装有Ceph且有MON服务
$ ssh root@server001 "sudo ceph config generate-minimal-conf" | sudo tee /etc/ceph/ceph.conf

# 添加权限
$ chmod 644 /etc/ceph/ceph.conf
```

#### 三、创建访问Ceph集群授权Key文件（就是客户端访问集群的Token文件，这个文件的内容我们事先应该在Ceph集群上创建好，以供客户端使用）
```bash
$ cat >>/etc/ceph/ceph.client.my_pool.keyring<<EOF
[client.my_pool]
        key = AQCdgKhhPxHKERAAA7TEsWOiY0wi9kwrc039Uw==
EOF  

# 添加权限
$ chmod 600 /etc/ceph/ceph.client.my_pool.keyring
```

#### 四、挂载 Ceph 文件系统
```bash
# 查看Ceph集群信息
# --name 指定用户名（注意：这个用户名必须在访问Ceph集群授权Key文件当中存在）
$ ceph --name client.my_pool -s

# 创建一个目录用于挂载Ceph文件系统
$ mkdir -p /home/mycephfs

# 挂载Ceph文件系统
# :/                   客户端目录挂载到Ceph文件系统的根目录/（注意：:号前面还可以指定Ceph集群MON节点地址和端口）
# /home/mycephfs 客户端要挂载Ceph文件系统的目录
# name           指定Ceph集群授权用户
# fs             指定Ceph文件系统名称（我们的Ceph集群可能创建了多个文件系统）
$ mount -t ceph :/ /home/mycephfs -o name=my_pool,fs=cephfs
```