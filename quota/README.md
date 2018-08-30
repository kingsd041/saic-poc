## 释放配额资源

平台内主要使用Deployment，验证当其副本数为0时是否自动重新刷新namespace的quota

### 创建基本资源

使用以下命令创建：`kubectl create -f .`

查看基本信息：

```
# kubectl get all -n test-namespace
NAME                                   READY     STATUS    RESTARTS   AGE
pod/nginx-deployment-94d9c6477-77vv4   1/1       Running   0          16s
pod/nginx-deployment-94d9c6477-tdbkd   1/1       Running   0          16s

NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2         2         2            2           16s

NAME                                         DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-deployment-94d9c6477   2         2         2         16s

# kubectl describe quota -n test-namespace
Name:            compute-resources
Namespace:       test-namespace
Resource         Used   Hard
--------         ----   ----
limits.cpu       500m   1
limits.memory    128Mi  1Gi
requests.cpu     500m   1
requests.memory  128Mi  1Gi
```

### 验证quota变化

调整副本数为0：`kubectl scale deployment nginx-deployment --replicas=0 -n test-namespace`

查看quota信息：

```
# kubectl describe quota -n test-namespace
Name:            compute-resources
Namespace:       test-namespace
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     1
limits.memory    0     1Gi
requests.cpu     0     1
requests.memory  0     1Gi
```

