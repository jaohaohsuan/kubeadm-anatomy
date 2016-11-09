FROM alpine:3.4

RUN apk add --no-cache --virtual build-deps wget && \
    wget --no-check-certificate -P / https://storage.googleapis.com/kubernetes-release-dev/ci-cross/v1.5.0-alpha.2.421+a6bea3d79b8bba/bin/linux/amd64/kubeadm && \
    chmod +x kubeadm && \
    apk del build-deps && \
    rm -rf /var/cache/apk/* 

ENV KUBE_HYPERKUBE_IMAGE gcr.io/google_containers/hyperkube-amd64:v1.4.5
ENV API_SERVER 192.168.60.136
ENV ETCD_ENDPOINTS http://192.168.60.136:2379

VOLUME ["/etc/kubernetes", "/opt/bin", "/etc/cni/net.d"]

CMD /kubeadm init --skip-preflight-checks --use-kubernetes-version v1.4.5 --api-advertise-addresses ${API_SERVER} --external-etcd-endpoints ${ETCD_ENDPOINTS}
