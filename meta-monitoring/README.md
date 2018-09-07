## meta-monitoring

Prometheus部署在业务k8s集群中（不会外置部署），它负责收集K8S workload和基础服务的一些监控信息。这里的元监控是指对Prometheus的监控，考虑使用Zabbix来实现。
Zabbix需要配置针对Prometheus的监控项，可包括：

1. Prometheus server所在节点Node的基本信息：CPU、内存、Disk使用情况（也需要考虑TSDB所在的存储磁盘）
2. Prometheus 主要服务的进程监控

### 自定义模板
模板名称 | 监控项表达式 | 触发器表达式 | 备注
---|---|---|---
Prometheus process | proc.num[,,all,alertmanager] | 	{Prometheus process:proc.num[,,all,alertmanager].max(#2)}<1 | 监控alartmanager进程
Prometheus process | proc.num[,,all,prometheus] | 	{Prometheus process:proc.num[,,all,prometheus].max(#2)}<1 | 监控prometheus进程
Prometheus process | proc.num[,,all,grafana-server] |{Prometheus process:proc.num[,,all,grafana-server].max(#2)}<2 | 监控grafana-server进程
Prometheus process | proc.num[,,all,grafana-watcher] |{Prometheus process:proc.num[,,all,grafana-watcher].max(#2)}<2 | 监控grafana-watcher进程
Prometheus process | proc.mem[prometheus] | {Prometheus process:proc.mem[prometheus].avg(#3)}>500M | prometheus进程占用内存
Prometheus process | proc.cpu.util[prometheus] | 	{Prometheus process:proc.cpu.util[prometheus].avg(#3)}>50 | prometheus进程占用cpu
Prometheus port | net.tcp.port[,9090] | {Prometheus port:net.tcp.port[,9090].last()}=0 | 监控prometheus server 端口
Prometheus web | web场景监控prometheus server | {Prometheus web:web.test.rspcode[prometheus metrics,prometheus_metric].last()}<>200 | prometheus server web监控
Opentsdb | net.tcp.port[,4242] | {Opentsdb:net.tcp.port[,4242].last()}=0 | 根据实际情况决定是否需要
Opentsdb | proc.num[,,,opentsdb] | {Opentsdb:proc.num[,,,opentsdb].max(#2)}<1 | 根据实际情况决定是否需要
Opentsdb | proc.mem[opentsdb] | 	{Opentsdb:proc.mem[opentsdb].last(#3)}>500M | 根据实际情况决定是否需要
Opentsdb | proc.cpu.util[opentsdb] | {Opentsdb:proc.cpu.util[opentsdb].last(#3)}>50 | 根据实际情况决定是否需要
Disk IO status | custom.vfs.dev.write.ops[xvda] |  | 磁盘写的次数
Disk IO status | custom.vfs.dev.write.ms[xvda] |  | 磁盘写的毫秒数
Disk IO status | custom.vfs.dev.write.sectors[xvda] | | 写扇区的次数（一个扇区的等于512B）
Disk IO status | custom.vfs.dev.read.ops[xvda] |  | 磁盘读的次数
Disk IO status | custom.vfs.dev.read.ms[xvda] |  | 磁盘读的毫秒数
Disk IO status | custom.vfs.dev.read.sectors[xvda] | | 读扇区的次数（一个扇区的等于512B）
High Write IOPS disk | device.write.ops[xvda] | {High Write IOPS disk:device.write.ops[xvda].min(20m)}>50 | 

### 如何使用

#### 模板导入 
1. 将所有模板(*.xml)通过zabbix web页面导入到zabbix中

#### 定制模板&Agent修改

1. 导入"Disk IO status"模板后，还需要在zabbix-agent配置文件中添加自定义脚本：

```
# cp Disk_IO_status_config/userparameter_io.conf /etc/zabbix/zabbix_agentd.d/userparameter_io.conf
# systemctl restart zabbix-agent
```
2. 导入"Disk_high_write_iops.xml"模板后，还需要在zabbix-agent配置文件中添加自定义脚本。如不需要自动发现，可在模板中禁用。
```
# cp hwdiskIOPS_config/hwdiskIOPS.conf /etc/zabbix/zabbix_agentd.d/hwdiskIOPS.conf
# cp hwdiskIOPS_config/hwIO.py /etc/zabbix/bin/hwIO.py
# systemctl restart zabbix-agent
```
3. 模板导入新环境后，需要根据实际情况修改:   
  (1) "Prometheus port"->"net.tcp.port[,9090]"和对应触发器的端口   
  (2) "Prometheus web"->"prometheus_metric" 对应的prometheus server url   
  (3) "High Write IOPS disk"->"device.write.ops[xvda]"对应的 TSDB 磁盘
  (4) "Disk IO status"->"custom.vfs.dev.write.xxx[xvda]" 对应的TSDB磁盘
### 其他
主机监控（CPU、内存、磁盘、网络）使用zabbix默认模板 `Template Module ICMP Ping`和 `Template OS Linux`

