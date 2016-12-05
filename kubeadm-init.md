# kubeadm init

## args

```
--token string  Shared secret used to secure cluster bootstrap; if none is provided, one will be generated for you
```
自己产生token
```
python -c 'import random; print "%0x.%0x" % (random.SystemRandom().getrandbits(3*8), random.SystemRandom().getrandbits(8*8))'
```

## api-advertise

这个IP地址很重要一旦设置错误, node也就无法join了, 所以这个IP地址必须与node之间可以互通.

```
kubeadm init --api-advertise-addresses="private_ip, $public_ip"
```


## CreateTokenAuthFile
从这个[源代码](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/tokens.go)可以看出创建token到`/etc/kubernetes/pki/tokens.csv`.

```
# cat /etc/kubernetes/pki/tokens.csv
80ea75.aa312981a8656e74,kubeadm-node-csr,b59f98a6-9818-11e6-ae22-56b6b6499611,system:kubelet-bootstrap
```

这里返现了一个新的群组名`system:kubelet-bootstrap`, 这个被用在

```
controller-manager
  --insecure-experimental-approve-all-kubelet-csrs-for-group=system:kubelet-bootstrap
```

此参数官方的意思
>The group for which the controller-manager will auto approve all CSR(certificate signing request)s for kubelet client certificates.

[参考](http://kubernetes.io/docs/admin/authentication/)Authenticating里面的`Static Token File`那段, 产生的内容, 格式`token,user,uid,"group1,group2,group3"`


## [WriteStaticPodManifests](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/manifests.go)

三个主要的Static Pods + ectd Pod

- kubeAPIServer
- kubeControllerManager
- kubeScheduler
- etcd (optional)

#### kube-apiserver
LivenessProbe: 8080, /healthz
cpu: 250m

##### kube-controller-manager
LivenessProbe: 10252, /healthz
cpu: 200m

##### kube-scheduler
LivenessProbe: 10251, /healthz
cpu: 100m

##### etcd (optioanl)
LivenessProbe: 2379, /healthz
cpu: 200m

## CreatePKIAssets

创建CA签名的双向证书，用于master和node上的各组件互相沟通.

[源代码](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/pki.go), 会生成以下证书

- `newCertificateAuthority` 产生`ca-key.pem`,`ca.pem`,`ca-pub.pem`
- `newServerKeyAndCert`     产生`apiserver-key.pem`,`apiserver.pem`,`apiserver-pub.pem`
- `newServiceAccountKey`    产生`sa-key.pem`, `sa-pub.pem`, 目前都没被使用

疑问？`ca.pem`为何还要转换成`ca-pub.pem`

## [CreateCertsAndConfigForClients](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/kubeconfig.go), [WriteKubeconfigIfNotExists](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/util/kubeconfig.go)

主要在`/etc/kubernetes/`下创建了`admin.conf`与`kubelet.conf`，

- 问题1 竟然是用`ca-key.pem`与`ca.pem`来做`client-certificate`与`client-key`
- 问题2 两个档案一模一样

## [CreateClientAndWaitForAPI](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/apiclient.go)

## [UpdateMasterRoleLabelsAndTaints](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/apiclient.go)

```
func attemptToUpdateMasterRoleLabelsAndTaints(client *clientset.Clientset, schedulable bool) error {
    n, err := findMyself(client)
    if err != nil {
        return err
    }

    # 打label
    n.ObjectMeta.Labels["kubeadm.alpha.kubernetes.io/role"] = "master"

    if !schedulable {
        # 可控制master上的kubelet不加入调度容器, 也就是说不成为node
        taintsAnnotation, _ := json.Marshal([]api.Taint{{Key: "dedicated", Value: "master", Effect: "NoSchedule"}})
        n.ObjectMeta.Annotations[api.TaintsAnnotationKey] = string(taintsAnnotation)
    }

    if _, err := client.Nodes().Update(n); err != nil {
        if apierrs.IsConflict(err) {
            fmt.Println("<master/apiclient> temporarily unable to update master node metadata due to conflict (will retry)")
            time.Sleep(apiCallRetryInterval)
            attemptToUpdateMasterRoleLabelsAndTaints(client, schedulable)
        } else {
            return err
        }
    }

    return nil
}
```

## CreateDiscoveryDeploymentAndSecret

TODO

## [CreateEssentialAddons](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/master/addons.go)

主要产生
- kube-proxy (DaemonSet)
- kube-dns (Deployment)

