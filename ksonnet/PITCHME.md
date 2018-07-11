## ksonnet 介绍

ksonnet is a framework for writing, sharing, and deploying Kubernetes application manifests. With its CLI, you can generate a complete application from scratch in only a few commands, or manage a complex system at scale.

使用Jsonnet language，JSON的超集，支持变量和对象组合。

---
### ksonnet 介绍
![Ksonnet](https://user-images.githubusercontent.com/14309137/42670601-77a667d0-868e-11e8-9235-33e66475cf79.png)

---
### 官方示例
![Application architecture](https://ksonnet.io/images/tutorial-img-1.png)

---
### 创建namespace环境
```
kubectl create namespace ks-dev

CURRENT_CONTEXT=$(kubectl config current-context)
CURRENT_CLUSTER=$(kubectl config get-contexts $CURRENT_CONTEXT | tail -1 | awk '{print $3}')
CURRENT_USER=$(kubectl config get-contexts $CURRENT_CONTEXT | tail -1 | awk '{print $4}')

kubectl config set-context ks-dev \
  --namespace ks-dev \
  --cluster $CURRENT_CLUSTER \
  --user $CURRENT_USER
```
@[1-2](创建namespace)
@[3-6](获取集群context)
@[7-10](命名context)

---
### 初始化应用
```
ks init guestbook --context ks-dev
```
![Directory structure](https://user-images.githubusercontent.com/14309137/42549658-ea495e72-84ff-11e8-88b5-3621b759a00a.png)

---
### 生成组件
```
ks generate deployed-service guestbook-ui \
  --image gcr.io/heptio-images/ks-guestbook-demo:0.1 \
  --type ClusterIP

ks show default
ks apply default
```
@[1-4](根据模板生成组件)
@[5](查看对应的yaml内容)
@[6](部署组件)

---
### 访问web服务
```
kubectl proxy
```
访问[guestbook](http://127.0.0.1:8001/api/v1/namespaces/ks-dev/services/guestbook-ui/proxy)
![Web ui](https://user-images.githubusercontent.com/14309137/42549843-e29525ac-8500-11e8-9a25-101039fcd4f2.png)

---
### 部署redis组件
```
ks pkg list
ks pkg install incubator/redis@master
ks prototype list
ks prototype describe redis-stateless
ks generate redis-stateless redis
ks show default -c redis
```
@[1](![pkg list](https://user-images.githubusercontent.com/14309137/42620825-a36fb768-85ee-11e8-897d-0d3190ac48ba.png))
@[2](安装组件包到本地)
@[3](![prototype list](https://user-images.githubusercontent.com/14309137/42620950-06c8c05c-85ef-11e8-80ed-61c4bbf7f04d.png))
@[4](![prototype desc](https://user-images.githubusercontent.com/14309137/42621016-2ab8e05a-85ef-11e8-9574-32a55fb161b8.png))
@[5](根据prototype生成组件)
@[6](查看组件对应的yaml内容)

---
### 多环境部署
```
kubectl create namespace ks-prod
kubectl config set-context ks-prod \
  --namespace ks-prod \
  --cluster $CURRENT_CLUSTER \
  --user $CURRENT_USER

ks env list
ks env add prod --context=ks-prod
ks env set default --name dev
ks env list

ks apply prod
```
@[1-6](设置新的k8s集群context)
@[7-11](查看ks环境context)
@[12](部署当前应用到prod环境)

---
### 按环境设置应用参数
```
ks param diff dev prod
ks param set guestbook-ui image gcr.io/heptio-images/ks-guestbook-demo:0.2 --env dev
ks param set guestbook-ui replicas 3 --env prod

ks apply dev && ks apply prod
ks diff remote:dev remote:prod
```
@[1](查看dev环境和prod环境参数差异)
@[2](dev环境设置image参数)
@[3](prod环境设置replicas参数)
@[5](部署到dev和prod环境)
@[6](已部署应用在dev和prod的差异)

---
### 删除部署
```
ks delete dev && ks delete prod
kubectl delete namespace ks-dev ks-prod
kubectl config delete-context ks-dev && kubectl config delete-context ks-prod
```
@[1](删除应用的部署)
@[2](删除k8s集群的namespace)
@[3](删除context命名)
