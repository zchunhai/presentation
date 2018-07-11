## kubeflow 介绍

Kubeflow致力于利用Kubernetes将机器学习（ML）工作流程的部署变得简单，便携和可扩展。
kubeflow作为ksonnet的registry，托管在github，包含多个package:

- JupyterHub
- TensorFlow Serving
- pytorch-job
- ...

---
### ksonnet 介绍
![Ksonnet](https://user-images.githubusercontent.com/14309137/42670601-77a667d0-868e-11e8-9235-33e66475cf79.png)

---
### kubeflow core 部署
```
kubectl create clusterrolebinding tf-admin --clusterrole=cluster-admin --serviceaccount=default:tf-job-operator
APP_NAME=my-kubeflow
ks init ${APP_NAME}
cd ${APP_NAME}
NAMESPACE=kubeflow
kubectl create namespace ${NAMESPACE}
ks env set default --namespace ${NAMESPACE}

VERSION=v0.2.0
ks registry add kubeflow github.com/kubeflow/kubeflow/tree/${VERSION}/kubeflow
ks pkg install kubeflow/core@${VERSION}
ks pkg install kubeflow/tf-serving@${VERSION}

ks generate core kubeflow-core --name=kubeflow-core
ks param set kubeflow-core cloud gke
ks param set kubeflow-core jupyterNotebookPVCMount /home/jovyan
ks param set kubeflow-core jupyterHubServiceType LoadBalancer
ks param set kubeflow-core tfAmbassadorServiceType LoadBalancer
ks param set kubeflow-core tfJobUiServiceType LoadBalancer
ks apply default -c kubeflow-core

kubectl delete crd tfjobs.kubeflow.org
ks delete default

```
@[1-8](创建环境)
@[9-13](安装组件)
@[14-21](生成模板并部署)
@[22-24](删除部署)

---
### kubeflow core 部署
```
kubectl get all -n kubeflow
kubectl port-forward tf-hub-0 8100:8000 -n kubeflow
```
![kubeflow all](https://user-images.githubusercontent.com/14309137/42556225-827f44f0-851d-11e8-98a1-93004a9f79ee.png)

---
### jupyterhub
![jupyterhub](https://user-images.githubusercontent.com/14309137/42556380-fdd3c306-851d-11e8-8784-104c628b0ed7.png)

---
### TensorFlow Serving

每个模型的部署都是一个ksonnet组件。
```
MODEL_COMPONENT=serveInception
MODEL_NAME=inception
MODEL_PATH=gs://kubeflow-models/inception
ks generate tf-serving ${MODEL_COMPONENT} --name=${MODEL_NAME}
ks param set ${MODEL_COMPONENT} modelPath ${MODEL_PATH}
ks apply default -c ${MODEL_COMPONENT}
```

---
### Seldon Serving
```
ks pkg install kubeflow/seldon@${VERSION}
ks generate seldon seldon
ks apply default -c seldon
```

---
### Submit TensorFlow training job

每个training都是一个ksonnet组件。
TFJob是kubeflow定义的k8s资源类型（CRD），用于简化分布式训练的编排和集群属性的配置。
```
ks pkg install kubeflow/examples@${VERSION}

CNN_JOB_NAME=mycnnjob
ks generate tf-job-simple ${CNN_JOB_NAME} --name=${CNN_JOB_NAME}
ks apply default -c ${CNN_JOB_NAME}
kubectl describe TFJob mycnnjob -n kubeflow
ks delete default -c ${CNN_JOB_NAME}
```

---
### Submit PyTorch training job
```
ks pkg install kubeflow/pytorch-job@${VERSION}
ks prototype use io.ksonnet.pkg.pytorch-operator pytorch-operator --name pytorch-operator
ks apply default -c pytorch-operator

JOB_NAME=myjob
ks generate pytorch-job ${JOB_NAME} --name=${JOB_NAME}
ks prototype describe pytorch-job # 查看参数列表
ks apply default -c ${JOB_NAME}
ks delete default -c ${JOB_NAME}
```
@[1-4](部署PyTorchJob CRD)
@[5-9](提交任务)
