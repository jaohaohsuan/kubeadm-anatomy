#Setup calico cni plugin for kubernetes

Step 1 adding ConfigMap that will be used in `/etc/cni/net.d`

```
ETCD_IP=192.168.60.136 envsubst < calico-config.yml | kubectl apply -f -
```

Setp 2 create calico-node on every node

```
kubectl apply -f calico-node-daemonset.yml
```

Step 3 create calico pool for pod CIDR configuration

```
kubectl apply -f configure-calico.yml
```
