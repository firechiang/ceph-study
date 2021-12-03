#### 注意：以下命令在装有Cephadm工具的节点上执行
```bash
# 列出所有警告或错误
$ ceph crash ls-new
ID                                                                ENTITY                NEW  
2021-12-03T03:37:43.643291Z_851e78db-be5b-4989-a105-a1a0fc94f2a5  mgr.server001.wgeiey   *   
2021-12-03T03:38:22.308618Z_de14cfb0-991e-482b-ad2a-2606a298a6a3  mgr.server001.wgeiey   * 

# 查看警告错误信息
$ ceph crash info <crash-id>

# 存档指定ID警告或错误信息（该命令执行完成以后，WEB管理界面将不会有该条警告或错误信息）
$ ceph crash archive <crash-id>

# 存档指定所有警告或错误信息（该命令执行完成以后，WEB管理界面将不会有该条警告或错误信息）
$ ceph crash archive-all

# 完全禁用WEB界面显示警告或错误（注意：不推荐使用）
$ ceph config set mgr/crash/warn_recent_interval 0
```