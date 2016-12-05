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

Step 4 create calico policy controller

```
kubectl apply -f calico-policy-controller.yml
```

## migrate from flannel

### step 1 config kubelet

Find your kubelet config file and add `--network-plugin=cni`, for example(vim /etc/default/kubelet).

Extract cni [tar.gz](https://storage.googleapis.com/kubernetes-release/network-plugins/cni-amd64-07a8a28637e97b22eb8dfe710eeae1344f69d16e.tar.gz) to `/opt/cni/bin` or `apt install kubernetes-cni`(from kubeadm [installation](http://kubernetes.io/docs/getting-started-guides/kubeadm/))

### step 2 remove docker flannel options

Find systemd docker drop-ins `systemctl cat docker`. It should be like below

```
# /lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target

# /etc/systemd/system/docker.service.d/flannel.conf
[Service]
EnvironmentFile=/run/docker_opts.env
```
and then `rm /etc/systemd/system/docker.service.d/flannel.conf`.

### step 3 disable and stop flannel

```
# systemctl disable flannel.service
# systemctl stop flannel.server
```
### step 4 reload & restart

```
# systemctl daemon-reload
# systemctl restart docker
# systemctl restart kubelet
```

