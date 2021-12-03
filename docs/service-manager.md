#### 集群服务管理相关（停止，启动，重启相关服务），要操控其它服务只需要修改服务名称即可（服务名称可在WEB控制界面Cluster > Services栏可以查看，也可使用 ceph -s 命令查看）（注意：以下命令在装有Cephadm工具的节点上执行）
```bash
# 启动所有mon服务
$ ceph orch restart mon

# 重启所有mon服务
$ ceph orch restart mon
Scheduled to restart mon.server001 on host 'server001'
Scheduled to restart mon.server002 on host 'server002'
Scheduled to restart mon.server003 on host 'server003'

# 重启某一个节点的mon服务
$ ceph orch restart mon.server001
```