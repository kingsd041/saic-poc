## portal-server

portal-server作为User Portal UI的后端服务，为了让前端调用各种API方便，需要portal-server来封装一些API，其中需要对接以下几种API：

1. C01 API
2. Logging API
3. Rancher API
4. K8S Native API

User Portal会做成一个iframe页面嵌入到C01中，整体架构如下：

![](https://ws2.sinaimg.cn/mw1024/006tNbRwgy1fuxfr7v0ywj31500gumxe.jpg)

### 关键功能点

portal-server采用Golang开发，API规范参考[Rancher API Spec](https://github.com/rancher/api-spec)。

#### 用户/租户/租户组

用户授权的打通需要Rancher SSO的支持，这部分会在Rancher中定制。

租户是C01中的概念，对应到Rancher的Project中，在这个客户的场景下，一个Project只有一个Namespace。
因为客户需要把各种隔离策略放在租户层级，而Rancher的所有策略设置都在Project层级（原生的k8s其实是Namespace层级），
所以这里为了保持体系能够对应，我们就设定一个Rancher Proejct中只有一个Namespace。

租户组是C01中的概念，就是多个租户的集合，一个租户只能属于一个租户组。

在初始访问User Portal时，UI需要调用后端创建Rancher Project（也就是租户）。

租户和租户组的关系可以调用C01 API获得。

#### 应用/应用组

应用是C01中的概念，与之对应就是创建k8s的Deployment。

应用组是C01中的概念，是一些应用的集合，只需在创建应用时候加入应用组的label。User Portal上可以依赖Label来筛选应用组。

#### 配额管理

C01上会对接多个k8s集群，配额是指一个租户在一个k8s上的配额。对应在Rancher概念下，就是一个Rancher Project的配额。
由于一个Project中只有一个Namespace，那么实际上也就是Namespace的配额。

C01中的配额有时会有更新（通常是加大配额），需要一种机制来保证配额上限信息能够同步到Rancher中。可以考虑在每次创建应用之前，同步一次配额上限。

每个应用的创建时都可以选择规格，主要是CPU和内存，对应的需要设置Pod的request&limit资源限制。创建成功后，一般Namespace的quota used信息会变化，
需要将quota used信息同步到C01的租户配额中。

风险点：

1. Rancher Project中还不支持设置配额，等待后续版本更新。或者考虑用k8s native API就把配额设置到Namespace上。

#### 日志服务

Rancher会把Project level和Application Level的日志都发送给第三方的Kafka集群，日志的聚合索引完全交给第三方服务。
UI上需要获取应用层面的日志（可能包括stdout和log file），portal-server提供API来帮助前端获取日志。
portal-server直接读取第三方日志服务接口即可，需要支持按时间和关键词检索。

#### 网络隔离

网络隔离包括两个层面：应用之间隔离和租户之间隔离。

应用之间隔离：

1. 默认情况下，租户内的应用不能互相访问
2. 选择了同一个网络策略的应用之间可以互相访问

租户之间隔离：

1. xxxxx

#### 其他聚合API

各种满足前端需求，可能需要聚合多个k8s/rancher API为一个API。
