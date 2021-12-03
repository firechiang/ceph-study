#### 客户端管理用户授权（建议：客户端管理用户不要给Ceph集群以外的节点使用）[官方中文文档](http://docs.ceph.org.cn/rados/operations/user-management/)（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 进入Ceph数据目录（为了方便管理，创建好的客户端授权文件就放到这里）
$ cd /etc/ceph

# 创建用户 client.my_pool（注意：实际用户名是叫my_pool而client是用户类型（表示客户端），这是约定俗成命名方式）
# 对mon组件有读取权限，对osd组件有读写执行权限但仅限于my_pool存储池（如果不指定存储池将对所有存储池有效）
# 最后将授权秘钥写到当前目录 ceph.client.my_pool.keyring 文件里面
$ ceph auth get-or-create client.my_pool mon 'allow r' osd 'allow rwx pool=my_pool' -o ceph.client.my_pool.keyring

# 查看用户client.my_pool授权信息（key就是授权秘钥）
$ ceph auth get client.my_pool
[client.my_pool]
  key = AQCPfqhh23XBCBAAzJ23aMUH48Kl3EKHn7Gbjg==
  caps mon = "allow r"
  caps osd = "allow rwx pool=my_pool"
  
# 修改用户client.my_pool相关权限
$ ceph auth caps client.my_pool mon 'allow r' osd 'allow rwx pool=my_pool'
  
# 删除用户 client.my_pool
$ ceph auth del client.my_pool
```

#### CephFs文件系统使用用户授权（要使用Ceph文件系统的用户使用如下授权）
```bash
# 创建用户 client.client_cephfs（注意：实际用户名是叫client_cephfs而client是用户类型（表示客户端），这是约定俗成命名方式）
# 授权对名字叫cephfs的文件系统的根目录 / 有 rw（读写权限），最后将授权秘钥写到当前目录 ceph.client.client_cephfs.keyring 文件里面
$ ceph fs authorize cephfs client.client_cephfs / rw -o ceph.client.client_cephfs.keyring

# 查看用户client.client_cephfs授权信息（key就是授权秘钥）
$ ceph auth get client.client_cephfs
exported keyring for client.client_cephfs
[client.client_cephfs]
  key = AQBr66lh/vWoDhAAIsXS3zuj7fKlwNB41kWsEQ==
  caps mds = "allow rw"
  caps mon = "allow r"
  caps osd = "allow rw tag cephfs data=cephfs"
  
# 删除用户 client.client_cephfs
$ ceph auth del client.client_cephfs
```
