---
title: "[Kubernetes In Action] Service & Ingress"

date: 2024-10-14T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Kubernetes"]
author: "Qingfeng Zhang"
---

# Service
## K8s中的Service是什么
先给出Service的定义：**K8s中的service是一种为一组功能相同的pod提供单一不变的接入点的资源**。

创建一个Service后，它的ip和port不会改变，K8s会为Service创建一个cluster ip来让集群内部的其他pod访问，也可以直接通过Service Name进行访问，K8s的DNS服务会将域名`service-name.namespace.svc.cluster.local`解析为cluster ip。

## 为什么需要Service
当我们在K8s中创建一些pod之后，这些pod可能是为集群内其他pod提供服务，也可能是为集群外部的请求提供服务。

按照一般的方法，如果想要访问一个服务，就需要知道它的ip和服务暴露的port，但是这种方式在K8s中不太适用，因为：
1. K8s中的pod可能会因为各种原因删除、新建或者重启，而每次新创建的pod会分配新的ip，并且ip分配是在pod启动之前执行的，这导致无法准确知道所需要访问的pod的准确ip
2. 可能会有多个pod同时提供一个服务，每个pod都有自己的ip，对于服务的使用者来说，不应该考虑使用哪一个pod，负载均衡的问题应该由K8s来解决

为了解决以上的问题，就需要**Service**这种K8s的资源类型。

## 如何使用Service
1. 可以使用`kubectl expose`命令创建Service，例如：
```shell
# Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
kubectl expose rc nginx --port=80 --target-port=8000
```
2. 通过编写yaml配置然后更新到K8s集群上，本质和`kubectl expose`是一样的，例如：
```yaml yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
  - port: 80
    targetport: 8080
  selector
    app: kubia
```
这将会创建一个名字为kubia的Service资源，这个Service会监听80端口的请求，并把请求转发到具有标签选择器app=kubia的pod的8080端口上。

Service创建之后K8s会为其分配一个cluster ip，可以用于访问Service。

> Service会自动配置负载均衡策略，如果想要指定访问某个pod的服务，可以配置Service的`sessionAffinity: ClientIP`，这会将来自同一个ClientIP的请求都转发到同一个pod

## Service的服务发现
我们为pod创建Service之后，就可以通过稳定的Service的IP来访问我们的多个pod提供的服务了，但是其他的pod又是怎么知道Service的IP和端口的呢？

K8s提供了两种Service服务发现的方式：
1. 通过环境变量

在pod创建完成刚开始运行的时候，K8s会创建一系列的环境变量指向现在的Service，可以在pod中查看ENV来列出这些环境变量。

这种方式的问题在于环境变量的创建是基于当前存在的Service，如果Service的创建在pod创建之后，这个pod的环境变量中就没有新创建的Service的信息。

2. 通过DNS

一般在K8s集群的kube-system这个namespace下会有名为core-DNS的pod，集群中的pod可以配置为使用其作为DNS（K8s可以修改pod的/etc/resolve.conf来实现），这样在pod上的DNS查询都会被coreDNS服务响应。

> 这就使得在集群内部可以使用全限定域名（FQDN）`service-name.namespace.svc.cluster.local`来访问K8s的Service，并且在同一个namespace中，只需要使用Service name访问即可

## 暴露Service到集群外部
很多时候我们不只是想在集群内部访问pod上的服务，某些服务需要提供给集群外部的人来访问，对此K8s提供了几种方式：
1. NodePort类型的Service

可以设置Service中spec的type为NodePort来创建此类型的Service，这会让K8s在每个Node上都保留一个相同的port用于Service的访问，因为Node的IP是外部可以访问的

2. LoadBalance类型的Service

LoadBalance是NodePort的扩展，在使用NodePort的时候虽然从不同的Node的IP+port访问的是同一个Service，但是Node会有多个，我们只能选择其中一个，因此LoadBalance就用于解决这个问题，通过负载均衡器（云厂商提供）提供一个外部可访问的External-IP来访问我们的Service，并实现Node的负载均衡

3. Ingress

另一种K8s资源类型，可以通过一个IP公开多个Service

Ingress运行在HTTP层（网络协议第七层），相比于Service（网络协议第四层）可提供的功能更多

> Service的类型：ClusterIP、NodePort、LoadBalance、ExternalIP

# Endpoint
Service配置中spec的selector会定义连接到哪些pod，但是实际上访问Service的时候不会把请求直接重定向到pod中

实际上K8s会把selector中对应的pod的IP和port存储在Endpoint资源中，当我们请求服务时，kube proxy会选择其中一个IP和port，并将请求重定向到该pod中

当我们意识到Endpoint是Service和Pod中间的另一种资源，用于进行解耦，我们就可以手动配置这个Service的Endpoint，例如不在Service中配置Selector，并手动修改其对应的Endpoint，实现使用K8s的Service将请求重定向到任意一个IP和port

# Ingress
## K8s中的Ingress是什么
前面也提到Ingress是一种可以向集群外部公开Service的K8s资源类型

## 为什么需要Ingress
如果只是想在集群外部访问集群内的服务，使用LoadBalance类型的Service是不是已经够了？但是如果我们有多个Service需要暴露出来呢，这样每个Service都要有自己的负载均衡器，以及External-IP，而Ingress只需要一个公网IP就可以为多个Service提供访问，并且因为是在第七层，可以接收HTTP请求，从而可以根据不同的URL参数访问不同的Service

Ingress的使用相对比较简单，后面还是通过Service-Endpoint-Pod的顺序，在Ingress的配置中也可以加上TLS来使用HTTPS
