# Way of using kubeadm

copy binaries to host

```
docker run -it --rm -v /opt/cni:/tmp/cni -v /opt/bin:/tmp/bin index.tenxcloud.com/henryrao/kubeadm:v1.4.5 /bin/sh -c "cp -r /opt/cni/bin /tmp/cni/ && cp /kubeadm /tmp/bin"
```
