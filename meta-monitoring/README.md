## meta-monitoring

Prometheus部署在业务k8s集群中（不会外置部署），它负责收集K8S workload和基础服务的一些监控信息。这里的元监控是指对Prometheus的监控，考虑使用Zabbix来实现。
Zabbix需要配置针对Prometheus的监控项，可包括：

1. Prometheus server所在节点Node的基本信息：CPU、内存、Disk使用情况（也需要考虑TSDB所在的存储磁盘）
2. Prometheus 主要服务的进程监控


### 自定义模板
模板名称 | 监控项表达式 | 触发器表达式
---|---|---
Prometheus process | proc.num[,,all,alertmanager] | 	{Prometheus process:proc.num[,,all,alertmanager].max(#2)}<1
Prometheus process | proc.num[,,all,prometheus] | 	{Prometheus process:proc.num[,,all,prometheus].max(#2)}<1
Prometheus process | proc.num[,,all,grafana-server] |{Prometheus process:proc.num[,,all,grafana-server].max(#2)}<2
Prometheus process | proc.num[,,all,grafana-watcher] |{Prometheus process:proc.num[,,all,grafana-watcher].max(#2)}<2
Prometheus process | proc.mem[prometheus] | {Prometheus process:proc.mem[prometheus].avg(#3)}>500M
Prometheus process | proc.cpu.util[prometheus] | 	{Prometheus process:proc.cpu.util[prometheus].avg(#3)}>50
Prometheus port | net.tcp.port[,32723] | {Prometheus port:net.tcp.port[,32723].last()}=0
Prometheus web | web场景监控prometheus server | {Prometheus web:web.test.rspcode[prometheus metrics,prometheus_metric].last()}<>200
Opentsdb | net.tcp.port[,4242] | {Opentsdb:net.tcp.port[,4242].last()}=0
Opentsdb | proc.num[,,,opentsdb] | {Opentsdb:proc.num[,,,opentsdb].max(#2)}<1
Opentsdb | proc.mem[opentsdb] | 	{Opentsdb:proc.mem[opentsdb].last(#3)}>500M
Opentsdb | proc.cpu.util[opentsdb] | {Opentsdb:proc.cpu.util[opentsdb].last(#3)}>50
### 其他模板
主机监控（CPU、内存、磁盘、网络）使用zabbix默认模板 `Template Module ICMP Ping`和 `Template OS Linux`
