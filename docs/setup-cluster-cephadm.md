#### 一、集群节点分布，[官方文档](https://ceph.readthedocs.io/en/latest/cephadm/install/)
```bash
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

#### 二、修改 [vi /etc/hosts] 集群每个节点的Host信息
```bash
192.168.78.129 server002
192.168.78.130 server003
192.168.78.131 server004

# 将Host信息分发到集群的各个节点
$ scp /etc/hosts root@server002:/etc/                                            
```

#### 三、关闭 selinux安装Python3和配置时间同步以及[安装Docker](https://github.com/firechiang/kubernetes-study/blob/master/docker/docs/docker-online-install.md)（注意：集群中每个节点都要操作）
```bash
$ cat /etc/selinux/config                                      # 查看selinux信息
$ sestatus                                                     # 查看selinux状态
$ sed -i "/^SELINUX/s/enforcing/disabled/" /etc/selinux/config # 永久关闭 selinux（重启机器生效）
$ setenforce 0                                                 # 临时关闭selinux(建议使用，安装好了集群再重启机器就开启了selinux)

$ systemctl status chronyd                                     # 查看时间同步工具是否启动
$ yum install -y chrony                                        # 安装时间同步工具（注意：Centos默认已经安装好了）
$ systemctl enable --now chronyd                               # 启动时间同步工具

$ yum install python3 -y                                       # 安装 Python3，因为很多Ceph脚本依赖Python3
$ python3 -V  
```

#### 四、在集群的某个节点上安装集群引导工具Cephadm
##### 4.1、安装集群引导工具Cephadm和添加Ceph软件源
```bash
# 下载集群引导脚本文件（注意：octopus 是Ceph的版本）
$ curl https://raw.githubusercontent.com/ceph/ceph/octopus/src/cephadm/cephadm -o cephadm
$ chmod +x cephadm

# 添加Ceph的软件源配置文件
$ ./cephadm add-repo --release octopus
```

##### 4.2、修改 [vi /etc/yum.repos.d/ceph.repo] Ceph软件源使用阿里云的地址
```bash
[Ceph]
name=Ceph x86_64
baseurl=https://mirrors.aliyun.com/ceph/rpm-octopus/el7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc

[Ceph-noarch]
name=Ceph noarch
baseurl=https://mirrors.aliyun.com/ceph/rpm-octopus/el7/noarch
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc

[Ceph-source]
name=Ceph SRPMS
baseurl=https://mirrors.aliyun.com/ceph/rpm-octopus/el7/SRPMS
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
```

##### 4.3、安装Cephadm和Ceph-Common包
```bash
# 安装 Cephadm（注意：在集群引导脚本文件所在目录执行）
$ ./cephadm install
INFO:cephadm:Installing packages ['cephadm']...

# 安装Ceph-Common包，里面包含了所有的ceph命令，其中包括ceph，rbd，mount.ceph（用于安装CephFS文件系统）
$ cephadm install ceph-common

# 验证 ceph-common 是否安装成功
$ ceph -v
ceph version 15.2.4 (7447c15c6ff58d7fce91843b705a268a1917325c) octopus (stable)

# 验证 cephadm 是否安装成功
$ cephadm --help
# 查看cephadm版本（注意：这个命令需要启动Docker，而且可能还要重启机器）
$ cephadm version
```

#### 五、使用Cephadm集群引导工具创建集群（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 创建Cephadm工作目录
$ mkdir -p /etc/ceph

# 创建集群的第一个Monitors(MON)节点，就是集群健康协调服务的第一个节点（注意：IP就填当前节点的ip）。该命令执行以下操作：
# 1，在本地主机上为新集群创建monitor 和 manager daemon守护程序。
# 2，为Ceph集群生成一个新的SSH密钥，并将其添加到root用户的/root/.ssh/authorized_keys文件中
# 3，将与新群集进行通信所需的最小配置文件保存到/etc/ceph/ceph.conf。
# 4，向/etc/ceph/ceph.client.admin.keyring写入client.admin管理（特权！）secret key的副本
# 5，将public key的副本写入/etc/ceph/ceph.pub
# cephadm bootstrap 命令使用说明可使用：cephadm bootstrap -h 命令查看
# --output-dir 指定数据存储目录
$ cephadm bootstrap --mon-ip 192.168.78.129
INFO:cephadm:Ceph Dashboard is now available at:
       # Ceph Dashboard访问地址密码（注意：首次访问需要修改密码）
	    URL: https://server002:8443/
	    User: admin
	Password: tgdofrsp24

INFO:cephadm:You can access the Ceph CLI with:

	sudo /usr/sbin/cephadm shell --fsid 4aff06ce-f261-11ea-bcc6-000c29356622 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

INFO:cephadm:Please consider enabling telemetry to help improve Ceph:

	ceph telemetry on

For more information see:

	https://docs.ceph.com/docs/master/mgr/telemetry/

INFO:cephadm:Bootstrap complete.

# 查看秘钥文件是否创建成功
$ ll /etc/ceph
ceph.client.admin.keyring  ceph.conf  ceph.pub

# 查看必要容器是否下载成功
$ docker images

# 查看已启动的服务
$ docker ps
CONTAINER ID   IMAGE                       COMMAND                 CREATED         STATUS       PORTS
# Ceph监控Web服务（监控数据展示dashboard）
8ed24fcaf2ac   ceph/ceph-grafana:latest    "/bin/sh -c 'grafana…"  3 minutes ago   Up 3 minutes 
# prometheus 监控报警服务       
65af1bb13921   prom/alertmanager:v0.20.0   "/bin/alertmanager -…"  3 minutes ago   Up 3 minutes
# prometheus 监控服务     
a54437ed9f0e   prom/prometheus:v2.18.1     "/bin/prometheus --c…"  4 minutes ago   Up 4 minutes 
# prometheus 本地监控服务（节点数据收集）            
a1613eeaefe9   prom/node-exporter:v0.18.1  "/bin/node_exporter …"  4 minutes ago   Up 4 minutes  
# Ceph-crash 服务（崩溃数据收集模块与monitor守护进程一起运行，并收集守护进程出现 crashdumps （崩溃）的信息，并将其存储在ceph集群中，以供以后分析）      
1576edf747ed   ceph/ceph:v15               "/usr/bin/ceph-crash…"  6 minutes ago   Up 6 minutes
# Ceph Manager(MGR)（管理程序与monitor守护进程一起运行，为外部监视和管理系统提供额外的监视和接口）
511d80fefee0   ceph/ceph:v15               "/usr/bin/ceph-mgr -…"  9 minutes ago   Up 9 minutes
# Ceph Monitors(MON)（Ceph监视器通过保存集群状态的映射来跟踪整个集群的健康状况，就是集群健康协调服务）     
c355a73eb2f6   ceph/ceph:v15               "/usr/bin/ceph-mon -…"  9 minutes ago   Up 9 minutes

# 查看Ceph版本（ 使用 ceph -v 命令也可以）
$ cephadm shell ceph -v
INFO:cephadm:Inferring fsid 4aff06ce-f261-11ea-bcc6-000c29356622
INFO:cephadm:Inferring config /var/lib/ceph/4aff06ce-f261-11ea-bcc6-000c29356622/mon.server006/config
INFO:cephadm:Using recent ceph image ceph/ceph:v15
ceph version 15.2.4 (7447c15c6ff58d7fce91843b705a268a1917325c) octopus (stable)

# 查看Ceph集群状态以及运行的组件（ 使用 ceph status 命令也可以）
$ cephadm shell ceph status
INFO:cephadm:Inferring fsid 4aff06ce-f261-11ea-bcc6-000c29356622
INFO:cephadm:Inferring config /var/lib/ceph/4aff06ce-f261-11ea-bcc6-000c29356622/mon.server006/config
INFO:cephadm:Using recent ceph image ceph/ceph:v15
  cluster:
    id:     4aff06ce-f261-11ea-bcc6-000c29356622
    health: HEALTH_WARN
            Reduced data availability: 1 pg inactive
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 1 daemons, quorum server006 (age 45m)
    mgr: server006.fhmiom(active, since 43m)
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     100.000% pgs unknown
             1 unknown
             
# 查看集群所有组件运行状态（注意：如果当前机器没有 Ceph Monitors(MON)集群健康协调服务，该命令将无法执行）         
$ ceph orch ps

# 查看集群 mds 组件运行状态（注意：如果当前机器没有 Ceph Monitors(MON)集群健康协调服务，该命令将无法执行）       
$ ceph orch ps --daemon-type mds        

# 查看集群 mgr 组件运行状态（注意：如果当前机器没有 Ceph Monitors(MON)集群健康协调服务，该命令将无法执行）       
$ ceph orch ps --daemon-type mgr             
             
# 开放两个Web界面的端口             
$ firewall-cmd --zone=public --add-port=3000/tcp --permanent
$ firewall-cmd --zone=public --add-port=8443/tcp --permanent

# Ceph 管理端界面（注意：用户名和密码在集群创建时就有提示，上去查即可）
$ https://192.168.83.143:8443/
# Ceph 监控端界面
$ https://192.168.83.143:3000/           
```

#### 六、添加节点到集群和删除节点（注意：以下命令在装有集群引导工具Cephadm和Ceph-Common的机器上执行，就是在集群引导机器上执行）
##### 6.1、添加节点到集群
```bash
# 将公钥拷贝到想要加入集群的机器上（这个就是实现免密登陆）
$ ssh-copy-id -f -i /etc/ceph/ceph.pub root@server003
$ ssh-copy-id -f -i /etc/ceph/ceph.pub root@server004

# 将节点加入到集群
# 注意：加入到集群之后，会自动在该机器上启动 Ceph Monitors(MON)集群健康协调服务（Ceph监视器通过保存集群状态的映射来跟踪整个集群的健康状况）    
# 注意：启动 Ceph Monitors(MON) 服务较慢可能要几分钟（可到对应的机器上使用 docker ps 命令查看 Ceph Monitors(MON) 服务是否启动）
$ ceph orch host add server003
Added host 'server003'
$ ceph orch host add server004
Added host 'server004'

# 查看集群的所有节点
$ ceph orch host ls

# 查看集群状态（注意：如果当前机器没有 Ceph Monitors(MON)集群健康协调服务，该命令将无法执行）
$ ceph status
  cluster:
    id:     c8febfce-f27a-11ea-a7a8-000c29356622
    health: HEALTH_WARN
            Reduced data availability: 1 pg inactive
            OSD count 0 < osd_pool_default_size 3
 
  services:
    # Ceph Monitors(MON) 服务在这个三个节点上都有启动了
    mon: 3 daemons, quorum server006,server007,server008 (age 4m)
    mgr: server006.uscvav(active, since 86m), standbys: server007.jotdcz
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     100.000% pgs unknown
             1 unknown               
```
##### 6.2、彻底删除集群中的某个节点
```bash
# 清除server002节点上所有守护进程，以方便后面删除
$ ceph orch host drain server002
# 查看osd（对象存储）组件删除状态     
$ ceph orch osd rm status  
# 查看server002上是否还有守护进程
$ ceph orch ps server002  
# 如果server002上已经没有守护进程，执行下面命令将其删除
$ ceph orch host rm server002  
```

##### 6.3 、如果主机离线且无法恢复，可以使用以下命令将其从集群中删除
```bash
$ ceph orch host rm server002 --offline --force
```

#### 七、指定Ceph Monitors(MON)集群健康协调服务（类似于Zookeeper）部署在哪几台机器上（注意：如果我们没有指定Ceph Monitors(MON)集群健康协调服务的部署规则。每一个新的节点加入集群都会默认在该节点上安装一个Ceph Monitors(MON)（集群健康协调服务）最多装5个，所以我们要指定哪些机器安装Ceph Monitors(MON)集群健康协调服务，一般部署基数个即可）
```bash
# 指定server006,server007,server008机器安装Ceph Monitors(MON)集群健康协调服务（注意：以下节点必须包含引导节点，否则以后引导节点将无法控制集群）
# 注意：以下命令在装有集群引导工具Cephadm和Ceph-Common的机器上执行，就是在集群引导机器上执行
$ ceph orch apply mon server006,server007,server008
Scheduled mon update...

# 查看集群所有组件运行状态（注意查看哪几台服务器上有Ceph Monitors(MON)集群健康协调服务）    
$ ceph orch ps
```

#### 八、部署Object Storage Device(OSD) 对象存储组件（注意：机器里面必须有一块已经创建好了主分区但没有子分区，没有格式化，没有挂载目录，空间大于5GB的硬盘。才能在该节点部署Object Storage Device(OSD) 对象存储组件。因为部署对象存储组件需要指定一块空硬盘，否则无法部署。以下命令在装有集群引导工具Cephadm和Ceph-Common的机器上执行，就是在集群引导机器上执行）
```bash
# 告诉Ceph可使用任何可用和未使用的存储设备
$ ceph orch apply osd --all-available-devices
Scheduled osd.all-available-devices update...


# 在server002上部署Object Storage Device(OSD)对象存储组件，数据存储在/dev/sdb空硬盘上
# 注意：每台机器每块硬盘只能执行一次这个命令否则报错
$ ceph orch daemon add osd server002:/dev/sdb

# 在server003上部署Object Storage Device(OSD)对象存储组件，数据存储在/dev/sdb空硬盘上
# 注意：每台机器每块硬盘只能执行一次这个命令否则报错
$ ceph orch daemon add osd server003:/dev/sdb

# 在server004上部署Object Storage Device(OSD)对象存储组件，数据存储在/dev/sdb空硬盘上
# 注意：每台机器每块硬盘只能执行一次这个命令否则报错
$ ceph orch daemon add osd server004:/dev/sdb


# 查看群集所有主机上的存储设备清单
$ ceph orch device ls
server002  /dev/sda  hdd   50.0G   False  LVM detected, locked, Insufficient space (<5GB) on vgs  
server003  /dev/sda  hdd   50.0G   False  LVM detected, Insufficient space (<5GB) on vgs, locked  
server004  /dev/sda  hdd   50.0G   False  LVM detected, Insufficient space (<5GB) on vgs, locked
server004  /dev/sdb  hdd   10.0G   False  LVM detected, Insufficient space (<5GB) on vgs, locked

# 查看集群所有osd（对象存储）磁盘树型结构
$ ceph osd tree
ID  CLASS  WEIGHT   TYPE NAME           STATUS  REWEIGHT  PRI-AFF
-1         0.02939  root default                                 
-3         0.00980      host server001                           
 0    hdd  0.00980          osd.0           up   1.00000  1.00000
-5         0.00980      host server002                           
 1    hdd  0.00980          osd.1           up   1.00000  1.00000
-7         0.00980      host server003                           
 2    hdd  0.00980          osd.2           up   1.00000  1.00000
```