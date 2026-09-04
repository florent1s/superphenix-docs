# Getting started

We'll start with a small but production-shaped deployment. This guide targets a single AZ in **hyperconverged mode** using the minimal hardware profile (3 nodes), with the **management plane on the same cluster**.

## Before you begin

Run every command in this guide from a **bootstrap host** that can reach the nodes on the cluster network. For this lab, that is typically a laptop or jump host **connected to the same switch** as the three servers.

### Tools

Install these on the bootstrap host:

- **talosctl** — required to generate configs, apply them, and bootstrap the cluster ([talosctl CLI reference](https://www.talos.dev/latest/reference/cli/))
- **Helm** — required to install the `superphenix-operator` chart ([Helm install](https://helm.sh/docs/intro/install/))
- **kubectl** (recommended) and **k9s** (optional) — useful to inspect the cluster and debug if something goes wrong

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
  # Control plane nodes also handle running normal workloads
  allowSchedulingOnControlPlanes: true

  network:
    # We install the CNI ourselves using ArgoCD
    cni:
      name: none

  # We install CoreDNS ourselves using ArgoCD, with custom parameters and high availability
  coreDNS:
    disabled: true
```

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

# Hyperconverged AZ on the same cluster (Local connection).
clusters:
  cluster-local:
    deploymentTopology: Hyperconverged
    region: lab
    availabilityZone: lab-a
    connection:
      mode: Local
    systemConfiguration:
      apps:
        kubeovn:
          helm:
            values:
              # Comma-separated IPs of the three control-plane nodes.
              # Required today; auto-detection is planned in a future release.
              masterNodes: "192.168.1.11,192.168.1.12,192.168.1.13"
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
          helm:
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
