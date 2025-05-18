---

title: K8S 集群搭建
created: 2025-05-17
tags:
    - K8S
    - Install

---

```bash
# Run the following commands on the master node
kubeadm init --apiserver-advertise-address=10.10.10.25 --kubernetes-version 1.29.15 --service-cidr=10.10.0.0/12 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all --cri-socket unix:///var/run/cri-dockerd.sock

# Run the following commands on the worker node
kubeadm join 10.10.10.25:6443 --token q8t96r.j1i2bye3fqtnjlh0 \
        --discovery-token-ca-cert-hash sha256:891432de1dc05f33948100a18bcad56df09010045e881dd5b619550790b3b6eb \
        --cri-socket unix:///var/run/cri-dockerd.sock
```
