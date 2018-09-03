## meta-monitoring

Prometheus部署在业务k8s集群中（不会外置部署），它负责收集K8S workload和基础服务的一些监控信息。这里的元监控是指对Prometheus的监控，考虑使用Zabbix来实现。
Zabbix需要配置针对Prometheus的监控项，可包括：

1. Prometheus server所在节点Node的基本信息：CPU、内存、Disk使用情况（也需要考虑TSDB所在的存储磁盘）
2. Prometheus 主要服务的进程监控
