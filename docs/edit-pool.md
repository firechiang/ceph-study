#### 一、创建存储池[官方中文文档](http://docs.ceph.org.cn/rados/operations/pools/)（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 查看集群当中所有存储池
$ ceph osd lspools
1 device_health_metrics（注意：这个存储池是集群创建时就自动创建好了的）
2 test_pool
3 my_pool

# 创建一个名称叫test_pool的存储池
$ ceph osd pool create test_pool
```

#### 二、修改存储池相关配置
```bash
# 设置test_pool存储池的副本数为2
$ ceph osd pool set test_pool size 2

# 设置test_pool存储池最多可存储10000个对象，最大存储容量为10240个字节，也就是10K大小（取消限制将值改为0即可）
$ ceph osd pool set-quota data max_objects 10000 max_bytes 10240

# 修改test_pool存储池的名称为test_pool1
$ ceph osd pool rename test_pool test_pool1
```

#### 三、创建删除存储池快照
```bash
# 为test_pool存储池创建快照，名字叫 test_snap
$ ceph osd pool mksnap test_pool test_snap

# 删除test_pool存储池的test_snap快照
$ ceph osd pool rmsnap test_pool test_snap
```

#### 四、删除存储池（注意：Ceph集群管理员默认是不能删除存储池的）
```bash
# 允许删除存储池
$ ceph config set mon mon_allow_pool_delete true

# 删除名字叫test_pool的存储池
# --yes-i-really-really-mean-it 表示强制删除
$ ceph osd pool delete test_pool test_pool 
```