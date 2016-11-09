FROM alpine:3.4

ENV CNI_RELEASE 07a8a28637e97b22eb8dfe710eeae1344f69d16e

RUN apk add --no-cache --virtual build-deps curl wget && \
    wget --no-check-certificate -P / https://storage.googleapis.com/kubernetes-release-dev/ci-cross/v1.5.0-alpha.2.421+a6bea3d79b8bba/bin/linux/amd64/kubeadm >/dev/null 2>&1 && \
    chmod +x kubeadm && \
    curl -sSL --fail --retry 5 \
    https://storage.googleapis.com/kubernetes-release/network-plugins/cni-adm64-${CNI_RELEASE}.tar.gz \
    | tar xz && \
    mkdir -p /opt/cni && \
    mv /bin /opt/cni/ && \
    apk del build-deps && \
    rm -rf /var/cache/apk/* 

ENV KUBE_HYPERKUBE_IMAGE gcr.io/google_containers/hyperkube-amd64:v1.4.5
ENV API_SERVER 192.168.60.136
ENV ETCD_ENDPOINTS http://192.168.60.136:2379

VOLUME ["/etc/kubernetes", "/opt/bin", "/etc/cni/net.d"]

CMD /kubeadm init --skip-preflight-checks --use-kubernetes-version v1.4.5 --api-advertise-addresses ${API_SERVER} --external-etcd-endpoints ${ETCD_ENDPOINTS}
