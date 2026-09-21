# Network configuration

## Node subnets

Worker nodes need to talk to each other and therefore need at least one shared subnet between all node groups. You can than add more subnets to certain node groups depending on your needs.

???+ note "Internal cluster CIDRs"
    By default KaaS clusters use the following IP ranges:

    1. Pod CIDR:
        - `10.0.0.0/12`
        - `fd00:100::/96`
    2. Service CIDR:
        - `10.96.0.0/16`
        - `fd00:100:ffff::/112`
    
    To avoid issues your node's subnet CIDR should not overlap with the pod CIDR.

## Network policies

When deploying a cluster you can choose between a set of pre-defined **network policies** to regulate ingress and egress of your nodes and controlplanes and ensure that clusters remain operationnal despite restrictive security groups.
This chapter details those default policies and explains how you can configure them according to your requirements.

### Default policies

#### Nodes

**Default ingress rule**

| Allowed sources | Ports | Comment |
|-----------------|-------|---------|
| Any             | Any   |         |

**Strict ingress rule**

| Allowed sources                 | Ports   | Comment |
|---------------------------------|---------|---------|
| Other nodes of the same cluster | Any     |         |
| Any                             | 80, 443 | HTTP/S  |

**Default egress rule**

| Allowed destinations | Ports | Comment |
|----------------------|-------|---------|
| Any                  | Any   |         |

**Strict egress rule**

| Allowed destinations            | Ports            | Comment      |
|---------------------------------|------------------|--------------|
| Other nodes of the same cluster | Any              |              |
| Any                             | 53               | DNS          |
| Any                             | 80, 443          | HTTP/S       |
| Any                             | 7442, 7443, 7444 | Controlplane |

#### Controlplane

**Default ingress rule**

| Allowed sources | Ports              | Comment                    |
|-----------------|--------------------|----------------------------|
| Any             | 7442, 7443, 7444   | Controlplane ingress       |
| Any             | 8133               | Internal konnectivity port |
| Any             | 8134, 10257, 10259 | K8s probes                 |

**Default egress rule**

| Allowed destinations                    | Ports | Comment |
|-----------------------------------------|-------|---------|
| Datastores, coreDNS, controlplane peers | Any   |         |

### Customize the configuration

Here is how you can control the `networkPolicies` for the worker nodes:

=== "Console"

    1. Access your cluster's page under **PaaS → Kubernetes**.
    2. Click **Options → Edit** (top right).
    3. Under **General Information → Cluster Network Policies → Workers Network Policies** you can set:
        - **Ingress** to *default*, *strict* or *none*.
        - **Egress** to *default*, *strict* or *none*.
    4. Navigate to the last tab (**Network Configuration**) and click **Update** (bottom right).

=== "GitOps"

    ```yaml
    my-cluster:
      workers:
        network:
          policies:
            ingress: default/strict/none
            egress: default/strict/none
    ```

    Full chart values: [sfs-kaas](https://github.com/super-phenix/superphenix/blob/main/components/dependencies/sfs-kaas/values.yaml).

Setting one of those parameters to `none` disables the corresponding networkpolicy rule, giving you full control over which connections should be let through.

You can also supplement those default rules by creating your own security groups to further regulate traffic to and from the cluster, here are some labels you can use to target the cluster components in your custom rules:

!!! warning "Known limitations"
    Currently this mechanism cannot be used to limit controlplane access to a specific range of public IPs.

1. Controlplane
  ```yaml
  superphenix.net/resourceEffectiveID: <SPXID of your cluster>
  superphenix.net/workloadClass: kaas-tenant-api-server
  ```
2. All worker nodes
  ```yaml
  cluster.x-k8s.io/cluster-name: <SPXID of your cluster>
  ```
3. Node group
  ```yaml
  cluster.x-k8s.io/cluster-name: <SPXID of your cluster> # Recommended as <poolName> might be used by other clusters in the same project
  superphenix.net/vmPool: <poolName>
  ```
