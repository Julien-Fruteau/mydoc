---
title: kubernetes
tags:
  - bare-metal
  - arm64
---


<!--toc:start-->
- [Disclaimer](#disclaimer)
- [Post OS install](#post-os-install)
- [Network configuration](#network-configuration)
- [CGroup](#cgroup)
  - [KubeletConfiguration](#kubeletconfiguration)
- [Container Runtime](#container-runtime)
  - [runc](#runc)
  - [CNI plugins](#cni-plugins)
  - [containerd config](#containerd-config)
- [CRICTL](#crictl)
- [Kubeadm](#kubeadm)
  - [Prereq](#prereq)
    - [Firewall](#firewall)
    - [Swap](#swap)
  - [packages](#packages)
  - [control plane](#control-plane)
  - [kubeadm init](#kubeadm-init)
- [Kube VIP](#kube-vip)
- [Calico](#calico)
- [PKI certificates and requirements](#pki-certificates-and-requirements)
  - [Distributing Self-Signed Certificates](#distributing-self-signed-certificates)
- [Nodes DNS](#nodes-dns)
- [Join](#join)
  - [Worker node](#worker-node)
<!--toc:end-->

## Disclaimer

Creating a kubernetes cluster on arm64 server using kubeadm may leave
few resources left for application to run.

This documentation should mostly be considered as training/tutorial man pages.

`k3s` kubernetes cluster may be preferred (faster setup and less resources
used by k8s cluster) if you're willing to spin up a cluster quickly on arm64,
and deploying application on it.

## Post OS install

```sh
sudo dpkg-configure locales
sudo hostnamectl set-hostname <NODE-FQDN>
```

> [docs](https://v1-30.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
>
## Network configuration

Enable IPv4 packet forwarding

```sh
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Verify that net.ipv4.ip_forward is set to 1 with:
sysctl net.ipv4.ip_forward
```

## CGroup

Control Groups are used to constrain resources allocated to processes
Used by kubelet and container runtime to **enforce** resource management

~cgroupfs driver~ -> systemd cgroup driver, as it is recommended when
systemd is the ini system

### KubeletConfiguration

> This need to be added to kubelet configuration in kubeadm init conf file. Check this [section](#kubeadm-init)

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
failSwapOn: false
memorySwap:
  swapBehavior: LimitedSwap
```

cgroup may not be active by default on arm64 os

```sh
sudo vim /boot/firmware/cmdline.txt
```

> ⚠️  Do NOT add line breaks ! This file should remain a single line ⚠️

Add at the end of the line the following:

```txt
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 cgroup_enable=hugetlb
```

```sh
sudo reboot
# check
cat /proc/mounts |grep cgroup
# cgroup2 /sys/fs/cgroup cgroup2 rw,nosuid,nodev,noexec,relatime 0 0
```

## Container Runtime

📢 Using systemd for cgroup driver enforce using systemd for the CRI

> [docs](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

```sh
# pick up a containerd-<VERSION>-<OS>-<ARCH>.tar.gz
wget https://github.com/containerd/containerd/releases/download/v2.0.2/containerd-2.0.2-linux-arm64.tar.gz

wget https://github.com/containerd/containerd/releases/download/v2.0.2/containerd-2.0.2-linux-arm64.tar.gz.sha256sum

# check
sha256sum -c containerd-2.0.2-linux-arm64.tar.gz.sha256sum
# containerd-2.0.2-linux-arm64.tar.gz: OK

sudo tar Cxzvf /usr/local containerd-2.0.2-linux-arm64.tar.gz
```

to start containerd via systemd, you should also download the containerd.service

```sh
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

### runc

> [docs](https://github.com/opencontainers/runc/releases)

```sh
# pick a release from https://github.com/opencontainers/runc/releases
wget https://github.com/opencontainers/runc/releases/download/v1.2.4/runc.arm64
sudo install -m 755 runc.arm64 /usr/local/sbin/runc
```

### CNI plugins

> [docs](https://github.com/containernetworking/plugins/releases)

```sh
# pick a release from  https://github.com/containernetworking/plugins/releases
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-arm64-v1.6.2.tgz
# sha256sum
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-arm64-v1.6.2.tgz.sha256

sha256sum -c cni-plugins-linux-arm64-v1.6.2.tgz.sha256
# cni-plugins-linux-arm64-v1.6.2.tgz: OK

sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-arm64-v1.6.2.tgz
```

### containerd config

Save the default configuration in `/etc/containerd/config.toml`:

```sh
sudo mkdir /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

add the following to ensure the CRI is handled by systemd

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

```sh
sudo systemctl restart containerd.service
```

## CRICTL

CLI and validation tools for Kubelet Container Runtime Interface (CRI) .

> [install docs](https://github.com/kubernetes-sigs/cri-tools/releases)

```sh
VERSION="v1.32.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-arm64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-arm64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-arm64.tar.gz
```

file : `/etc/crictl.yaml`

```yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: false
pull-image-on-create: false
```

> [usage docs](https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/)

## Kubeadm

> [docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### Prereq

#### Firewall

```sh
sudo apt install netcat-openbsd dnsutils
# listen on port
nc -l 6443 &
# check port opened
nc -zv 127.0.0.1 6443
```

Full list of ports to be opened : [docs](https://kubernetes.io/fr/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
> if necessary enable and start `nftables.service` to configure the
least permissive firewall settings

#### Swap

- If possible disable swap, `sudo swapoff -a`

```txt
- Identify configured swap devices and files with cat /proc/swaps.
- Turn off all swap devices and files with swapoff -a.
- Remove any matching reference found in /etc/fstab.
- Optional: Destroy any swap devices or files found in step 1 to prevent their reuse.
Due to your concerns about leaking sensitive information, you may wish to consider
performing some sort of secure wipe.
```

- Otherwise tolerate swap :
  - `NodeSwap` must be enabled on the kubelet
  - `failSwapOn: false` in kubelet configuration

❓ kubelet configuration

```yaml
memorySwap:
  swapBehavior: LimitedSwap
```

### packages

kubeadm, kubelet, kubectl

- kubelet and kubeadm version should match
- kubelet <= any kube-apiserver
- kube-apiserver in HA cluster : 1 minor version skew max
- 3 minor version skew max than kube-apiserver < kube-proxy <= kube-apiserver

kubeadm creates etcd is in `/var/lib/etcd`. should be backed-up

```sh
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
# If the directory `/etc/apt/keyrings` does not exist,
# it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```

### control plane

To upgrade a single control-plane kubeadm cluster to high availability you should
specify the `--control-plane-endpoint` to set the shared endpoint for
all control-plane nodes.

### kubeadm init

```sh
kubeadm config print init-defaults
```

- kubeadm configuration migration : `kubeadm config migrate`:

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: InitConfiguration
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef  # 📢 replace this
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
localAPIEndpoint:
  advertiseAddress: <IP-Node>     # 📢 change me
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  name: <Node-Hostname>           # 📢 change me
  taints: null
timeouts:
  controlPlaneComponentHealthCheck: 4m0s
  discovery: 5m0s
  etcdAPICall: 2m0s
  kubeletHealthCheck: 4m0s
  kubernetesAPICall: 1m0s
  tlsBootstrap: 5m0s
  upgradeManifests: 5m0s
---
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
caCertificateValidityPeriod: 87600h0m0s
certificateValidityPeriod: 8760h0m0s
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
encryptionAlgorithm: RSA-2048
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.k8s.io
kubernetesVersion: 1.32.0
controlPlaneEndpoint: "<KUBE-VIP-HOST:6443" # 📢 change me
networking:
  dnsDomain: cluster.local        # 📢 change me
  # ~serviceSubnet: 10.96.0.0/12~
  serviceSubnet: 10.116.0.0/16    # 💡 update serviceSubnet and add podSubnet
  podSubnet: 10.117.0.0/16

proxy: {}
apiServer: {}
scheduler: {}
```

- append the custom config for the kubelet configuration:

```yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
failSwapOn: false
memorySwap:
  swapBehavior: LimitedSwap
```

- upload this config `init-config-m1.yaml` into the 1st control plane node

> 📢 the kube-vip cannot be used at 1st start, hence the ip must be temporarily
assigned to the network interface  of the node :

```sh
sudo ip add <KUBE-VIP-IP> dev <interface-name> # eth0 for instance, do ip link to check
```

- then run init :

```sh
sudo kubeadm init --config init-config-m1.yaml --upload-certs
```

🔥 Troubleshoot:

```sh
sudo kubeadm reset
sudo systemctl stop kubelet.service
# update the node setup (config, etc...)
# then run kubeadmn init command again ✈️
```

## PKI certificates and requirements

[best-practices](https://kubernetes.io/docs/setup/best-practices/certificates/)
[gen certs](https://kubernetes.io/docs/tasks/administer-cluster/certificates/)

Authentication over TLS : kubeadm can provide them auto, or generate your own certificates
in order not to store them on the API server

Certificate requirements:

- Server certificates:
  - for the API server endpoint
  - for the etcd server
  - for each kubelet
  - optional for the front-proxy

- Client certificates:
  - for  each kubelet, to auth to the API server as a client of the k8s API
  - for each API server, used to authenticate to etcd
  - control manager to securely communicate with the API server
  - scheduler to securely communicate with the API server
  - for each node, for kube-proxy to auth to the API server
  - optional for admins to auth to the API server
  - optional for the front-proxy

- storage :  `/etc/kubernetes/pki`

### Distributing Self-Signed Certificates

A client node may refuse to recognize a self-signed CA certificate as valid.
For a non-production deployment, or for a deployment that runs behind a company firewall,
you can distribute a self-signed CA certificate to all clients and refresh the local
list for valid certificates.

```sh
sudo cp ca.crt /usr/local/share/ca-certificates/kubernetes.crt
sudo update-ca-certificates
```

## Nodes DNS

On arm64, NetworkManager service may be used to configure `/etc/resolv.conf` file.

- to manually configure the dns:

`sudo vim /etc/NetworkManager/NetworkManager.conf`

```conf
[main]
dns=none
```

```sh
sudo systemctl restart NetworkManager.service
```

- now the file /etc/resolv.conf is empty, edit the file to your preferences (add a dns server)
- when rebooting networking, the `/etc/resolv.conf` file will stay as reconfigured

## Join

### Worker node

```sh
kubeadm join <CLUSTER-DOMAIN>:6443 --config <join-config.yaml>
```

- example join config:

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: <CLUSTER-DOMAIN>:6443   # 📢 change me
    caCertHashes:
    - sha256:****                              # 📢 change me
    token: ****                                # 📢 change me
  tlsBootstrapToken: ****                      # 📢 change me
kind: JoinConfiguration
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  name: <NODE-FQDN>                            # 📢 change me
  taints: null
timeouts:
  controlPlaneComponentHealthCheck: 4m0s
  discovery: 5m0s
  etcdAPICall: 2m0s
  kubeletHealthCheck: 4m0s
  kubernetesAPICall: 1m0s
  tlsBootstrap: 5m0s
  upgradeManifests: 5m0s
```

## Kube VIP

Doc: [kube-vip](kube-vip.md)

## Calico

Doc: [calico](calico.md)

🔥 Troubleshoot:

- check logs of the operator in namespace `tigera-operator`

## Nodes taint

⚠️  Taints are mostly handled by kubernetes, so most of the time it should not
be configured manually

- Nodes taints are handled by labels `node-role/kubernetes.io/...`, visible with a `kubectl describe nodes <node-name>`
- check `kubectl taint --help`, particularly for possible effects (NoSchedule, NoExecute (evict running pods), PreferNoSchedule)

```sh
# Add a taint, though set auto by the control plane
kubectl taint nodes <node-name> node.kubernetes.io/not-ready:NoExecute
# NOTE: this one tells to not schedule workload (application) on this node,
# you must add a toleration on the pods manifest to allow it to be scheduled 
# on this control-plane node (typcaly for system pods)
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint nodes <node-name> node.kubernetes.io/not-ready:NoSchedule
# Remove a taint with '-' at the end
kubectl taint nodes <node-name> node-role.kubernetes.io/...:NoSchedule-
```

## Node NotReady

- describe pods, `Conditions Type` might indicate valuable information, such as error with the CRI => restart service containerd
