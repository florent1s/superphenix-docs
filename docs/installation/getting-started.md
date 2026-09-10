# Getting started

We'll start with a small but production-shaped deployment. This guide targets a single AZ in **hyperconverged mode** using the minimal hardware profile (3 nodes), with the **management plane on the same cluster**.

## Before you begin

Run every command in this guide from a **bootstrap host** that can reach the nodes on the cluster network. For this lab, that is typically a laptop or jump host **connected to the same switch** as the three servers.

### Tools

Install these on the bootstrap host:

- **talosctl** — required to generate configs, apply them, and bootstrap the cluster ([talosctl CLI reference](https://www.talos.dev/latest/reference/cli/))
- **Helm** — required to install the `superphenix-operator` chart ([Helm install](https://helm.sh/docs/intro/install/))
- **kubectl** (recommended) and **k9s** (optional) — useful to inspect the cluster and debug if something goes wrong

### Domain names

A **domain name** is required to expose:

- The **web console**
- **ArgoCD**, if you expose it
- Services such as Kubernetes control planes, databases, and similar HTTP(S) endpoints

The cluster listens on **every node** on ports **80** and **443**. DNS only needs to point at **one** node. In a high-availability setup, put a **load balancer** in front of the nodes or use **DNS round-robin**.

If you do not have DNS, a wildcard service such as **[nip.io](https://nip.io)** can map a name to the cluster’s internal IP. Use that for labs that are not reachable from the internet.

### Planning

Review architecture planning documents so topology and infrastructure constraints are clear up front:

- [Architecture overview](../architecture/index.md)
- [Deployment topology](../architecture/deployment-topology.md)
- [Hardware requirements](../architecture/deployment-requirements.md)
- [Network requirements](../architecture/network-requirements.md)

For complete and advanced deployment paths, see the [deployment guide](deployment-guide/index.md).
For production-oriented configuration and performance guidance, see [Production recommendations](production-recommendations.md).

!!! warning "Official support scope"
    Superphenix can technically run on any conformant Kubernetes cluster, but we **officially support Talos**. Current default values and operational assumptions are tuned for Talos-based clusters.

## Reference lab setup

Use this baseline for a first deployment:

- **Topology**: Single AZ, hyperconverged
- **Management**: On the same cluster (`connection.mode: Local`)
- **Nodes**: 3 control-plane nodes (minimal specs)
- **Network**: Flat VLAN on a single switch

## Install Talos

!!! warning "Maximum supported Talos version"
    The maximum supported Talos version is **1.12.6**. Do not install a newer release.
    This is due to a Linux kernel bug affecting the SDN.

### Bootstrap Kubernetes

Quick reference:

- Generate cluster config:
  `talosctl gen config spx-local https://<api-endpoint>:6443`
- Apply machine configs to each node:
  `talosctl apply-config --insecure --nodes <node-ip> --file controlplane.yaml`
- Bootstrap etcd once from one control-plane node:
  `talosctl bootstrap --nodes <first-control-plane-ip>`

Official references:

- [Talos Getting Started](https://talos.dev/v1.11/introduction/getting-started)
- [talosctl CLI reference](https://www.talos.dev/latest/reference/cli/)

### Required Talos configuration

Add the following under `machine:` on **every** node before you apply the configs. It enables CDI device ownership for KubeVirt and loads the Open vSwitch kernel module used by Kube-OVN:

```yaml
machine:
  # Static file to override the containerd config
  # https://github.com/kubevirt/containerized-data-importer/issues/2378#issuecomment-1297007860
  files:
    - content: |
        [plugins]
          [plugins."io.containerd.grpc.v1.cri"]
            device_ownership_from_security_context = true
          [plugins."io.containerd.cri.v1.runtime"]
            device_ownership_from_security_context = true
      path: /etc/cri/conf.d/20-customization.part
      op: create

  kernel:
    modules:
      - name: openvswitch
```

Also set the following under `cluster:` in the generated configs. Superphenix installs the CNI and CoreDNS itself (via ArgoCD), and this 3-node lab schedules workloads on the control planes:

```yaml
cluster:
  # Control plane nodes also handle running normal workloads (this is a small cluster)
  allowSchedulingOnControlPlanes: true

  network:
    # The CNI is installed by Superphenix
    cni:
      name: none

  # CoreDNS is installed by Superphenix
  coreDNS:
    disabled: true
```

### External interface name

The interface that connects the cluster to the exterior must have the **same name on every node**. Superphenix refers to that name cluster-wide. It can be a **physical NIC**, a **bond**, or a **VLAN**.

If the kernel names differ between nodes, add a **link alias** so every node exposes the same name. Append a `LinkAliasConfig` document to each node's machine configuration. Match the NIC by MAC address, and use the same alias on every node:

```yaml
---
apiVersion: v1alpha1
kind: LinkAliasConfig
name: ext0
selector:
  match: mac(link.permanent_addr) == "00:1a:2b:3c:4d:5e" # this node's NIC MAC
```

Pick an alias that does not look like a kernel name (`eth0`, `ens3`, `enp0s31f6`, …), or it may conflict with a real interface. Change only the MAC per node; keep `name: ext0` identical.

Bonds and VLANs are named when you define them, so give them the same `interface` name on every node instead of an alias. Talos can alias **physical** links only.

### DHCP and default gateway

If node addresses come from DHCP, enable DHCP explicitly on each interface in the Talos machine configuration, and **set a default gateway on every node**. Without a default gateway, Kube-OVN can fail and break cluster connectivity.

```yaml
machine:
  network:
    interfaces:
      - interface: eth0 # replace with your interface name
        dhcp: true
        routes:
          - network: 0.0.0.0/0
            gateway: 192.168.1.1 # your lab default gateway
```

Apply the same pattern (with the correct gateway) on all three nodes.

## Install the operator

Create a values file (for example `values.yaml`) that installs the operator on this cluster, configures the management stack, and declares a **hyperconverged** `Cluster` with `connection.mode: Local` so Superphenix runs on the same Kubernetes cluster that hosts the operator.

Full chart reference: [superphenix-operator `values.yaml`](https://github.com/super-phenix/superphenix/blob/main/components/system/superphenix-operator/values.yaml).  
System / Kube-OVN defaults: [superphenix-system `values.yaml`](https://github.com/super-phenix/superphenix/blob/main/components/system/superphenix-system/values.yaml).

### Example values file

```yaml
# Required while bootstrapping: the Talos cluster has no CNI until Superphenix
# installs Kube-OVN. Set back to false after the stack is healthy.
installOnClusterWithoutCNI: true

# ArgoCD config to debug and see the deployments live
# Not required, but always useful to have
config:
  argocd:
    values:
      server:
        ingress:
          hostname: "argocd.example.org" # For internal IPs: argocd.192.168.1.10.nip.io

# Management stack configuration (including the console)
management:
  manual: true
  systemConfiguration:
    apps:
      kratos:
        values:
          globalSecret: "a-very-long-string-you-need-to-change"
      superphenix-console:
        values:
          # The domain on which the console will be hosted
          # For internal IPs: argocd.192.168.1.10.nip.io
          domain: console.example.org

# Hyperconverged AZ on the same cluster (Local connection).
clusters:
  local:
    deploymentTopology: Hyperconverged
    region: lab
    availabilityZone: lab-a
    connection:
      mode: Local
    systemConfiguration:
      apps:
        misc:
          values:
            objects:
              external-subnet:
                spec:
                  protocol: IPv4
                  cidrBlock: 192.168.1.0/24
                  gateway: 192.168.1.254
                  excludeIps:
                    - 192.168.1.0..192.168.1.200
                    - 192.168.1.254
              external-subnet-nad:
                spec:
                  config: '{
                      "cniVersion": "0.3.0",
                      "type": "macvlan",
                      "master": "ext0", # Put the name of your external interface here
                      "mode": "bridge",
                      "ipam": {
                        "type": "kube-ovn",
                        "server_socket": "/run/openvswitch/kube-ovn-daemon.sock",
                        "provider": "external-subnet.kube-system"
                      }
                    }'
        kubeovn:
          values:
            masterNodes: MASTER_1,MASTER_2,MASTER_3 # 192.168.1.150,192.168.1.151,192.168.1.153
```

### Install with Helm

```bash
helm install superphenix-operator \
  ghcr.io/super-phenix/charts/superphenix-operator \
  --namespace superphenix-system \
  --create-namespace \
  -f values.yaml
```

### Avoid CIDR conflicts with Kube-OVN

The node CIDR (the subnet of your Talos node IPs) must **not** overlap Kube-OVN networks from the system chart defaults:

| Network | Default (IPv4) | Default (IPv6) | Purpose |
|---------|----------------|----------------|---------|
| Pods | `10.0.0.0/12` | `fd00:100:0000:0::/96` | Pod overlay |
| Services | `10.16.0.0/12` | `fd00:100:ffff:0::/112` | ClusterIP services |
| Join | `100.64.0.0/12` | `fd00:100:64::/112` | Kube-OVN join network |
| Isolated egress | `10.32.0.0/16` | `fd00:110::/64` | System egress subnet |

Prefer choosing a node subnet that does not collide with those defaults. If you must keep an existing lab addressing plan that conflicts, override the CIDRs under `systemConfiguration.apps.kubeovn.helm.values` (IPv4, IPv6, or both).

Example when nodes are on `10.1.0.0/24` (inside the default pod range `10.0.0.0/12`):

```yaml
clusters:
  cluster-local:
    # ... same fields as above ...
    systemConfiguration:
      apps:
        kubeovn:
          values:
            masterNodes: "10.1.0.11,10.1.0.12,10.1.0.13"
            networking:
              pods:
                cidr:
                  v4: "10.128.0.0/12"
                  v6: "fd00:200:0000:0::/96"
                gateways:
                  v4: "10.128.0.1"
                  v6: "fd00:200:0000:0::1"
              services:
                cidr:
                  v4: "10.144.0.0/12"
                  v6: "fd00:200:ffff:0::/112"
              join:
                cidr:
                  v4: "100.64.0.0/12"
                  v6: "fd00:200:64::/112"
            # Isolated egress lives under extraObjects in the system chart;
            # only change it if your node CIDR also conflicts with
            # 10.32.0.0/16 or fd00:110::/64.
```
