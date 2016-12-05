# kubeadm anatomy

## 动机
为了将kubernetes部署在任何平台上，很有必要跟踪官方具体部署方式，kubeadm是个简化部署的工具，但不想不明不白的就这么装好了.

## [release](https://github.com/kubernetes/release)

安装包, 以`ubuntu 16.04`作为例子

- kubeadm [bin](https://github.com/kubernetes/release/blob/master/debian/xenial/kubeadm/debian/rules#L13)
- kubectl
- kubelet
- kubernetes-cni [bin](https://github.com/kubernetes/release/blob/master/debian/xenial/kubernetes-cni/debian/rules)

## Table of Contents
- [kubeadm init](kubeadm-init.md)
- [kubeadm join]()

## preflight

[检查项目](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/preflight/checks.go)

## images

依赖的[images](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/images/images.go)
 
manifests或addons
- gcr.io/google_containers/kube-discovery-amd64:1.0

## ENV
[这里](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/client-go/pkg/apis/kubeadm/env.go)可以看出kubernetes把档案都放到那里去了

- `/etc/kubernetes` 配置都在这个目录下
- `/etc/kubernetes/pki` node如何加入master的凭证token都在这里
- `/var/lib/etcd` kubernetes的数据库etcd

告诉/etc/kubernetes/manifests下的static Pods使用hyperkube image

```
KUBE_HYPERKUBE_IMAGE=gcr.io/google_containers/hyperkube
```

## [kube-discovery](https://github.com/kubernetes/kubernetes/tree/master/cluster/images/kube-discovery)
master顺带启动的一个新服务, 用于`kubeadm join`过程，这里也存了`join`时的token查看方式如下：
```
# disc_pod=`kubectl get po -l name=kube-discovery --namespace=kube-system --no-headers | awk '{print $1}'`
# kubectl exec -it ${disc_pod} --namespace=kube-system cat /tmp/secret/token-map.json
```

结果像这样

```
{"fd87cc":"bb32064ed38191bb"}
```

这就是我们`join`时用的`fd88cc.bb32444ed38191bb`token

## hyperkube

[官方image](https://github.com/kubernetes/kubernetes/tree/master/cluster/images/hyperkube)
