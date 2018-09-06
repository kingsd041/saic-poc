## meta-monitoring

Prometheus部署在业务k8s集群中（不会外置部署），它负责收集K8S workload和基础服务的一些监控信息。这里的元监控是指对Prometheus的监控，考虑使用Zabbix来实现。
Zabbix需要配置针对Prometheus的监控项，可包括：

1. Prometheus server所在节点Node的基本信息：CPU、内存、Disk使用情况（也需要考虑TSDB所在的存储磁盘）
2. Prometheus 主要服务的进程监控

### 自定义模板
模板名称 | 监控项表达式 | 触发器表达式 | 备注
---|---|---|---
Prometheus process | proc.num[,,all,alertmanager] | 	{Prometheus process:proc.num[,,all,alertmanager].max(#2)}<1
Prometheus process | proc.num[,,all,prometheus] | 	{Prometheus process:proc.num[,,all,prometheus].max(#2)}<1
Prometheus process | proc.num[,,all,grafana-server] |{Prometheus process:proc.num[,,all,grafana-server].max(#2)}<2
Prometheus process | proc.num[,,all,grafana-watcher] |{Prometheus process:proc.num[,,all,grafana-watcher].max(#2)}<2
Prometheus process | proc.mem[prometheus] | {Prometheus process:proc.mem[prometheus].avg(#3)}>500M
Prometheus process | proc.cpu.util[prometheus] | 	{Prometheus process:proc.cpu.util[prometheus].avg(#3)}>50
Prometheus port | net.tcp.port[,9090] | {Prometheus port:net.tcp.port[,9090].last()}=0
Prometheus web | web场景监控prometheus server | {Prometheus web:web.test.rspcode[prometheus metrics,prometheus_metric].last()}<>200
Opentsdb | net.tcp.port[,4242] | {Opentsdb:net.tcp.port[,4242].last()}=0 | 可能不需要
Opentsdb | proc.num[,,,opentsdb] | {Opentsdb:proc.num[,,,opentsdb].max(#2)}<1 | 可能不需要
Opentsdb | proc.mem[opentsdb] | 	{Opentsdb:proc.mem[opentsdb].last(#3)}>500M | 可能不需要
Opentsdb | proc.cpu.util[opentsdb] | {Opentsdb:proc.cpu.util[opentsdb].last(#3)}>50 | 可能不需要
Disk IO status | custom.vfs.dev.write.ops[xvda] |  | 磁盘写的次数
Disk IO status | custom.vfs.dev.write.ms[xvda] |  | 磁盘写的毫秒数
Disk IO status | custom.vfs.dev.write.sectors[xvda] | | 写扇区的次数（一个扇区的等于512B）
Disk IO status | custom.vfs.dev.read.ops[xvda] |  | 磁盘读的次数
Disk IO status | custom.vfs.dev.read.ms[xvda] |  | 磁盘读的毫秒数
Disk IO status | custom.vfs.dev.read.sectors[xvda] | | 读扇区的次数（一个扇区的等于512B）

### 其他模板
主机监控（CPU、内存、磁盘、网络）使用zabbix默认模板 `Template Module ICMP Ping`和 `Template OS Linux`


### 备注
1. 模板导入新环境后，需要根据实际情况修改"Prometheus port"->"net.tcp.port[,9090]"和对应触发器的端口   
2. 模板导入新环境后，需要根据实际情况修改"Prometheus web"->"	prometheus_metric" 对应的prometheus server url   
3. 导入"Disk IO status"模板后，还需要在zabbix-agent配置文件中添加自定义脚本：
添加userparameter_io.conf配置文件
- 在/etc/zabbix/zabbix_agentd.d下添加userparameter_io.conf, 文件内容如下:  
```
UserParameter=custom.vfs.dev.read.ops[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$4}'
UserParameter=custom.vfs.dev.read.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$7}'
UserParameter=custom.vfs.dev.write.ops[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$8}'
UserParameter=custom.vfs.dev.write.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$11}'
UserParameter=custom.vfs.dev.io.active[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$12}'
UserParameter=custom.vfs.dev.io.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$13}' 
UserParameter=custom.vfs.dev.read.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$6}'            
UserParameter=custom.vfs.dev.write.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$10}'          
```
- 重启zabbix-agent服务, `systemctl restart zabbix-agent`


