--- 
title: kube-vip
tags:
  - kubernetes
  - kube-vip
---

Provides a VIP address to ensure HA of control plane nodes,
and load balancing for services

## Use with kubeadm

[docs static pod](https://kube-vip.io/docs/installation/static/)

- update `VIP-IP-Address` and run following command:

```sh
export VIP=<VIP-IP-Address>
export INTERFACE=eth0
export KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
alias kube-vip="docker run --network host --rm ghcr.io/kube-vip/kube-vip:$KVVERSION"

kube-vip manifest pod \
    --interface $INTERFACE \
    --address $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection > kube-vip.yaml
```

Copy kube-vip.yaml on all masters : `/etc/kubernetes/manifest/kube-vip.yaml`.

📢 : for the 1st kubeadm init, the vip must be configure once : `ip addr add <IP-VIP>/32 dev <INTERFACE>`
