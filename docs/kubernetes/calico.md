---
title: calico
tags:
  - kubernetes
  - calico
---

[Docs](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)

Installation using tigera operator:

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml
```

Download the custom resources necessary to configure Calico:

```sh
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O
```

Edit `custom-resources.yaml`:

```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    nodeAddressAutodetectionV4:
      canReach: <NETWORK-GATEWAY-IP>      # 📢 change me
    ipPools:
    # # ℹ️  default conf:
    # - name: default-ipv4-ippool
    #   blockSize: 26
    #   cidr: 192.168.0.0/16
    #   encapsulation: VXLANCrossSubnet
    #   natOutgoing: Enabled
    #   nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```

Create the manifest to install Calico.

```sh
kubectl create -f custom-resources.yaml
```

Verify Calico installation in your cluster:

```sh
watch kubectl get pods -n calico-system
```

## Encapsulation Types in Calico

> One of: IPIP, VXLAN, IPIPCrossSubnet, VXLANCrossSubnet, None

- IPIP (Default for IPv4)

      Uses IP-in-IP (IPIP) encapsulation for all inter-node traffic.
      Required when nodes are in different subnets without direct Layer 2 connectivity.
      Pros: Low overhead, widely supported.
      Cons: Adds some encapsulation overhead, may require kernel module support.

- VXLAN

      Uses VXLAN encapsulation for all inter-node traffic.
      Works well in environments where IPIP is blocked or unsupported.
      Suitable for cloud providers like AWS, GCP, and OpenStack.
      Pros: More widely supported in cloud environments, allows for larger network scalability.
      Cons: Higher overhead than IPIP.

- IPIPCrossSubnet

      Uses IPIP encapsulation only when nodes are in different subnets.
      Directly routes traffic within the same subnet without encapsulation.
      Use Case: Optimized for environments where nodes within the same subnet can communicate directly but need encapsulation across subnets.

- VXLANCrossSubnet

      Uses VXLAN encapsulation only when nodes are in different subnets.
      Directly routes traffic within the same subnet.
      Use Case: Similar to IPIPCrossSubnet, but uses VXLAN instead.

- None

      No encapsulation at all.
      Requires direct L2 (Layer 2) connectivity between all nodes.
      Use Case: Ideal for on-premises deployments with proper routing or BGP-based setups.
