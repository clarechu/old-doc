---
layout: k8s-doc
title: 回滚部署
catalog: true
date: 2018-11-10 15:37:24
tags: 
- k8s
- deployment
subtitle: "回滚到历史版本"
header-img: "7vLMRNQ.jpg"
---

#

有时您可能想要回滚部署; 例如，当部署不稳定时，例如崩溃循环。默认情况下，所有Deployment的卷展栏历史记录都保留在系统中，以便您可以随时回滚（可以通过修改修订历史记录限制来更改）。

## 创建多个版本的 POD

镜像版本：

```shell
REPOSITORY                                                     TAG                 IMAGE ID            CREATED             SIZE
docker.vpclub.cn/hidevopsio/hinode-java-jar                    0.1                 bce3d8ba6ba1        5 days ago          728 MB
docker.vpclub.cn/hidevopsio/hinode-java-jar                    1.0                 00de2c65d80e        2 weeks ago         761.6 MB
```

分别是0.1 和 1.0 版本。

## 部署 v1版本的 hello-world

现在 在 namespace 为`demo` 创建 app 为`hello-world`的`pod`， 镜像为`docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1`。

```shell
# 获取当前的pod
kubectl get pod -n demo
NAME                          READY     STATUS    RESTARTS   AGE
hello-world-bbd694d8b-9pdcs   1/1       Running   0          8m
hello-world-bbd694d8b-djv2h   1/1       Running   0          8m

# 当pod STATUS为 RUNNING 的时候 pod为成功

```

* 获取pod的详细信息

```shell
kubectl describe pod hello-world-bbd694d8b-9pdcs -n demo
```

下面是 pod `hello-world-bbd694d8b-9pdcs`的基本信息，从下面的信息可以得到我们现  demo 组下面运行的pod为镜像名称为：docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1， sha256:`8907956124918ae37b6e53fa0c66c7be77e904785ba78f6154303d2b655709ab`

```yaml

Name:           hello-world-bbd694d8b-9pdcs
Namespace:      demo
Node:           k8s-06.k8s.local/172.16.10.46
Start Time:     Wed, 31 Oct 2018 11:47:20 +0800
Labels:         app=hello-world
                pod-template-hash=668250846
Annotations:    <none>
Status:         Running
IP:             10.244.0.42
Controlled By:  ReplicaSet/hello-world-bbd694d8b
Containers:
  hello-world:
    Container ID:   docker://5e4043acbe18304bd6c3b326ed3fc044b3cb4ec6f54f2a27e7254e5782bf406a
    Image:          docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1
    Image ID:       docker-pullable://docker.vpclub.cn/hidevopsio/hinode-java-jar@sha256:8907956124918ae37b6e53fa0c66c7be77e904785ba78f6154303d2b655709ab
    Port:           80/TCP
    State:          Running
      Started:      Wed, 31 Oct 2018 11:47:47 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dgqj2 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dgqj2:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dgqj2
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                       Message
  ----    ------                 ----  ----                       -------
  Normal  Scheduled              14m   default-scheduler          Successfully assigned hello-world-bbd694d8b-9pdcs to k8s-06.k8s.local
  Normal  SuccessfulMountVolume  14m   kubelet, k8s-06.k8s.local  MountVolume.SetUp succeeded for volume "default-token-dgqj2"
  Normal  Pulling                14m   kubelet, k8s-06.k8s.local  pulling image "docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1"
  Normal  Pulled                 14m   kubelet, k8s-06.k8s.local  Successfully pulled image "docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1"
  Normal  Created                14m   kubelet, k8s-06.k8s.local  Created container
  Normal  Started                14m   kubelet, k8s-06.k8s.local  Started container
```

## 部署v2版本的hello-world

方法如上：

获取v2版本的详细信息

```shell

kubectl get pod -n demo

NAME                           READY     STATUS    RESTARTS   AGE
hello-world-75d9cb465d-79vl8   1/1       Running   0          2m
hello-world-75d9cb465d-rbv2g   1/1       Running   0          2m
```

```shell
##获取pod的详细信息
kubectl describe pod hello-world-75d9cb465d-79vl8 -n demo
该pod的版本为 docker.vpclub.cn/hidevopsio/hinode-java-jar:1.0
sha256 ：c940617d09f231af1e902ad18315a8435b0220aef251fd7cab323bbca34e2166
```

如下：

```yaml
Name:           hello-world-75d9cb465d-79vl8
Namespace:      demo
Node:           k8s-04.k8s.local/172.16.10.44
Start Time:     Wed, 31 Oct 2018 12:57:52 +0800
Labels:         app=hello-world
                pod-template-hash=3185760218
Annotations:    <none>
Status:         Running
IP:             10.244.2.43
Controlled By:  ReplicaSet/hello-world-75d9cb465d
Containers:
  hello-world:
    Container ID:   docker://4a115dff91073e151c41e6a3b5fd4a6e514afb8853accc76b0678a669af81c65
    Image:          docker.vpclub.cn/hidevopsio/hinode-java-jar:1.0
    Image ID:       docker-pullable://docker.vpclub.cn/hidevopsio/hinode-java-jar@sha256:c940617d09f231af1e902ad18315a8435b0220aef251fd7cab323bbca34e2166
    Port:           80/TCP
    State:          Running
      Started:      Wed, 31 Oct 2018 13:00:21 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dgqj2 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dgqj2:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dgqj2
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                       Message
  ----    ------                 ----  ----                       -------
  Normal  Scheduled              2m    default-scheduler          Successfully assigned hello-world-75d9cb465d-79vl8 to k8s-04.k8s.local
  Normal  SuccessfulMountVolume  2m    kubelet, k8s-04.k8s.local  MountVolume.SetUp succeeded for volume "default-token-dgqj2"
  Normal  Pulling                2m    kubelet, k8s-04.k8s.local  pulling image "docker.vpclub.cn/hidevopsio/hinode-java-jar:1.0"
  Normal  Pulled                 15s   kubelet, k8s-04.k8s.local  Successfully pulled image "docker.vpclub.cn/hidevopsio/hinode-java-jar:1.0"
  Normal  Created                13s   kubelet, k8s-04.k8s.local  Created container
  Normal  Started                12s   kubelet, k8s-04.k8s.local  Started container
```

* 由详细信息可以看出该pod已经为最新的pod

现在我们需要回退到0.1版本 镜像那么我们怎么处理呢？

### 检查部署的部署历史记录

首先，检查此部署的修订版：

```shell
kubectl rollout history deployment/hello-world -n demo

## REVISION 为 deployment/hello-world 所有版本。

## CHANGE-CAUSE 为版本的注释

deployments "hello-world"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>

如果查看到的结果为None，则需要在创建和更新deployment时加上 --record参数.

```

修改注释

```shell
kubectl annotate deployment/hello-world kubernetes.io/change-cause="image 1.0" -n demo --revision=3
```

要进一步查看每个修订的详细信息，请运行：

```shell
kubectl rollout history deployment/hello-world -n demo --revision=2
```

## 回滚到以前的版本

现在您已决定撤消当前的卷展栏并回滚到以前的版本：

```shell
kubectl rollout undo deployment/hello-world -n demo

```

执行成功后pod会重新启动

```shell
kubectl get pod -n demo
NAME                           READY     STATUS              RESTARTS   AGE
hello-world-75d9cb465d-rbv2g   1/1       Running             0          21m
hello-world-bbd694d8b-gfh6x    1/1       Running             0          19s
hello-world-bbd694d8b-lvw88    0/1       ContainerCreating   0          18s
```

获取pod的相信信息, 由下面的例子我们可以看出镜像已经回退到上一个版本的 `docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1` sha256 : `8907956124918ae37b6e53fa0c66c7be77e904785ba78f6154303d2b655709ab`

```yaml
Name:           hello-world-bbd694d8b-gfh6x
Namespace:      demo
Node:           k8s-06.k8s.local/172.16.10.46
Start Time:     Wed, 31 Oct 2018 13:19:30 +0800
Labels:         app=hello-world
                pod-template-hash=668250846
Annotations:    <none>
Status:         Running
IP:             10.244.0.43
Controlled By:  ReplicaSet/hello-world-bbd694d8b
Containers:
  hello-world:
    Container ID:   docker://b88bcf11ef2c5439511f457baf29c0724d19dca1db7dbb560f7d13a475e1cf1d
    Image:          docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1
    Image ID:       docker-pullable://docker.vpclub.cn/hidevopsio/hinode-java-jar@sha256:8907956124918ae37b6e53fa0c66c7be77e904785ba78f6154303d2b655709ab
    Port:           80/TCP
    State:          Running
      Started:      Wed, 31 Oct 2018 13:19:48 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dgqj2 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dgqj2:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dgqj2
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                       Message
  ----    ------                 ----  ----                       -------
  Normal  Scheduled              2m    default-scheduler          Successfully assigned hello-world-bbd694d8b-gfh6x to k8s-06.k8s.local
  Normal  SuccessfulMountVolume  2m    kubelet, k8s-06.k8s.local  MountVolume.SetUp succeeded for volume "default-token-dgqj2"
  Normal  Pulling                2m    kubelet, k8s-06.k8s.local  pulling image "docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1"
  Normal  Pulled                 2m    kubelet, k8s-06.k8s.local  Successfully pulled image "docker.vpclub.cn/hidevopsio/hinode-java-jar:0.1"
  Normal  Created                2m    kubelet, k8s-06.k8s.local  Created container
  Normal  Started                2m    kubelet, k8s-06.k8s.local  Started container
```

或者，您可以通过在--to-revision以下位置指定回滚到特定修订：

```shell
kubectl rollout undo deployment/hello-world -n demo --to-revision=2
```

### 暂停和恢复部署

刚刚创建的部署, 使用该命令暂停：

```shell
kubectl rollout pause deployment\hello-world -n demo
```

暂停之前部署的初始状态将继续其功能，但只要部署暂停，部署的新更新将不会产生任何影响。

最后，恢复部署并观察一个新的ReplicaSet，提供所有新的更新：

```shell
kubectl rollout resume deployment/helo-world -n demo