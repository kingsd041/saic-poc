## load-balancing

负载均衡包括L4和L7的支持，K8S本身有相关的概念支撑，但是考虑到用户的网络结构和使用原则，按照以下方式支持：

1. L4和L7入口都通过LVS，使用DR（也可能是NAT）模式，DNS注册VIP的A记录。
2. L4/L7在k8s层面都通过ingress暴露，默认为L7代理80/443
3. 自定义controller watch ingress configmap数据（tcp/udp），并刷新LVS配置

大致的架构图如下：

![](https://ws2.sinaimg.cn/mw1024/0069RVTdly1fuwrr3zvwgj31f60pcq3d.jpg)

