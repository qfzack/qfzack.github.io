---
title: "记一次redis-operator的恢复"
date: 2025-04-22T05:00:00Z
# thumbnail: "images/photo/R0005812.JPG"
categories: ["Kubernetes"]
author: "Qingfeng Zhang"
---

## 1.问题背景

下午刚进完PR完成了version bump并升级线上的helm chart，发现有些service没有起来，仔细看下才发现redis服务刚好挂了，而且是个比较奇怪的问题导致pod起不来，而没有起来的service是配置服务，需要redis的数据，而其他的service又依赖于这个配置服务，进而导致服务挂了一片

> 其实这个配置服务不应该完全依赖redis，要能重建缓存，也可能是有我还没发现的细节

## 2.临时方案

需要先让redis起来，其他的服务才能够正常启动，redis的问题可以后面再排查，这里有考虑了几个点：

1. 之前的redis服务使用了PVC，并且使用RDB方式进行了持久化，数据放在/data路径下，为避免数据丢失，先对PV做一个volumeSnapshot

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: redis-data-snapshot
spec:
  volumeSnapshotClassName: <snapclass> #避免敏感信息
  source:
    persistentVolumeClaimName: redis-pvc
```

2. 创建一个redis pod来挂载这个PVC并以RDB方式启动来重建缓存数据

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
  labels:
    app: redis
spec:
  containers:
  - name: redis
    image: redis:7
    command: [ "redis-server", "--appendonly", "no" ]
    volumeMounts:
    - name: redis-data
      mountPath: /data
  volumes:
  - name: redis-data
    persistentVolumeClaim:
      claimName: redis-pvc
```

> 其实这里用deployment更好，pod失败可以自动重启

3. 因为集群内部是通过DNS对redis服务进行访问的，因此再创建一个service来指向这个pod

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
  type: ClusterIP
```

这样就可以使用`redis.<namespace>.svc.cluster.local:6379`来连接redis服务，并且保证这个连接和之前一样，不用修改其他服务的配置

这样一来我的服务都恢复了，但是这只是一个临时方案，我任然需要一个redis-operator来提供更可靠的redis服务

## 3.问题排查

原来redis pod的报错已经很清楚了，在redis配置文件的第17行指定的REDIS-PORT的值应该是6379，但是实际的值带有IP，这导致redis服务无法启动

然后找到这个环境变量的设置是在docker image内的一个脚本里，但是这个服务的配置已经很久没有更新了，而这个问题是最近突然出现的，就比较奇怪

## 4.服务恢复

### 4.1.redis升级部署

我使用的[redis operator](https://github.com/OT-CONTAINER-KIT/redis-operator)的版本有更新的，于是再创建一个namespace，在这个新的namespace里来部署最新版本的redis-operator

部署的过程在github repo中已经写的很清楚了，我使用的是standalone模式，使用helm安装redis-manager来在k8s集群上安装CRD、RBAC和operator，然后再用helm安装redis来创建CR的实例，这样起来的redis的数据是空的

### 4.2.redis数据恢复

原来的redis数据还在PVC中，在原来的namespace创建一个相同的工具pod使用相同的PVC，启动之后的工具pod有相同的redis数据：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
  labels:
    app: redis
spec:
  containers:
  - name: redis
    image: redis:7
    command: [ "redis-server", "--appendonly", "no" ]
    volumeMounts:
    - name: redis-data
      mountPath: /data
  volumes:
  - name: redis-data
    persistentVolumeClaim:
      claimName: redis-pvc
```

然后从这个pod中导出redis的rdb格式的数据到本地：

```bash
kubectl cp redis-recover:/data/dump.rdb ./dump-from-snapshot.rdb -n redis-old
```

接下来在新的namespace中创建一个相同的工具pod，并挂载和新的redis服务相同的PVC，这里使用的PV还是空的，然后导出的RDB数据通过工具pod导入到新的redis PVC中：

```bash
kubectl cp ./dump-from-snapshot.rdb redis-recover:/data/dump.rdb -n redis-new
```

不出意外的话重启新的redis服务让其读取RBD的数据，然后删除工具pod就完事了，但是出现了意外：*redis operator的cotroller逻辑是自己写的并且没有提供配置持久化方式的参数，并且默认的持久化方式是AOF，刚才导入的rdb数据无法生效*

这里即使手动修改了redis的持久化方式，在重启pod之后还是会失效

> 不确定是否支持配置RDB，至少在helm的配置里没有看到

于是前面导出的RDB数据没用了，进到旧的namespace的工具pod里，使用redis-cli查看和更改持久化的方式为AOF：

```bash
redis-cli -a Sm4rtcl0ud! CONFIG GET appendonly
redis-cli -a Sm4rtcl0ud! CONFIG SET appendonly no
redis-cli -a Sm4rtcl0ud! CONFIG SET appendonly yes
```

这样工具pod的redis就会重新把数据写到一个AOF文件中，然后再导出这个文件并导入到新的namespace的工具pod中：

```bash
kubectl cp redis-recover:/data/appendonly.aof ./appendonly.aof -n redis-old
kubectl cp ./appendonly.aof redis-recover:/data/appendonly.aof -n redis-new
```

> 也有相关的工具可以实现redis的AOF文件和RDB文件之间的转换

因为是和工具pod使用相同的PVC，所以新的redis服务可以读取到相同的AOF文件挂载在/data下

接下来再重启新的redis pod就会读取并加载AOF文件的数据到内存中了，这样新的redis服务就配置好了
