#### 一、安装Ceph客户端
```bash
# 查看Linux内核是否大于2.6（只要内核大于2.6就默认支持块设备）
$ uname -r
# 查看系统是否支持modprobe命令
$ modprobe rbd

# 安装Ceph客户端（注意：安装Ceph客户端需要添加Ceph软件源，软件源如何添加文档上面有）
$ yum install ceph -y
```
#### 二、创建Ceph配置文件，就是指定集群健康协调服务的各个节点，相当于指定Zookeeper集群（可以直接将Ceph集群配置文件拷贝过来，文件是Ceph节点Ceph数据目录ceph.conf文件）
```bash
# fsid 是Ceph集群ID，mon_host 是Ceph Monitors(MON)集群健康协调服务的各个节点
$ cat >>/etc/ceph/ceph.conf<<EOF
[global]
        fsid = 92ba74e6-528a-11ec-afbc-52540011ed8e
        mon_host = [v2:192.168.122.253:3300/0,v1:192.168.122.253:6789/0]
EOF
```

#### 三、创建访问Ceph集群授权Key文件（就是客户端访问集群的Token文件，这个文件的内容我们在上面已经创建好了）
```bash
$ cat >>/etc/ceph/ceph.client.my_pool.keyring<<EOF
[client.my_pool]
        key = AQCdgKhhPxHKERAAA7TEsWOiY0wi9kwrc039Uw==
EOF  
```

#### 四、挂载 Ceph 文件系统
```bash
# 查看Ceph集群信息
$ ceph --name client.my_pool -s
```