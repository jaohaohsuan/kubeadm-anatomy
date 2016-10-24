# kubeadm anatomy

## 动机
为了将kubernetes部署在任何平台上，很有必要跟踪官方具体部署方式，kubeadm是个简化部署的工具，但不想不明不白的就这么装好了.

## Table of Contents
- [kubeadm init](kubeadm-init.md)
- [kubeadm join]()

## ENV
[这里](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/client-go/pkg/apis/kubeadm/env.go)可以看出kubernetes把档案都放到那里去了

- `/etc/kubernetes` 配置都在这个目录下
- `/etc/kubernetes/pki` node如何加入master的凭证token都在这里
- `/var/lib/etcd` kubernetes的数据库etcd

告诉/etc/kubernetes/manifests下的static Pods使用hyperkube image

```
HYPERKUBE_IMAGE=gcr.io/google_containers/hyperkube
```

discovery_image (TODO)