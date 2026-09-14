# Using local storage

Superphenix normally uses Ceph to provide replicated network storage. For debugging on hardware with slow disks, Ceph replication and network overhead can make storage perform poorly. You can instead enable the local path provisioner and store volumes directly on each node.

!!! danger "Debugging only"
    Local storage is not resilient. Losing the node or its disk means losing the data stored there. A workload using a local volume is also tied to that node and cannot be rescheduled elsewhere while retaining its data. Do not use this configuration for production data.

## Enable local storage

Enable `local-path-provisioner` in the cluster's system configuration. For the local hyperconverged cluster configured through the `superphenix-operator` Helm values, use:

```yaml
clusters:
  local:
    systemConfiguration:
      apps:
        local-path-provisioner:
          enabled: true
```

## Disable Ceph and use only local storage

For a disposable debugging cluster, you can disable Ceph cluster and make `local-path` the default storage class. The local path provisioner chart uses `storageClass.defaultClass` for this setting:

```yaml
clusters:
  local:
    systemConfiguration:
      apps:
        rook-operator:
          enabled: false
        rook-local-cluster:
          enabled: false
        rook-connection:
          enabled: false
        local-path-provisioner:
          enabled: true
          values:
            storageClass:
              defaultClass: true
```

With this configuration, persistent volume claims that do not specify `storageClassName` use `local-path`.

!!! danger "Do not switch a cluster that contains data"
    Disabling Ceph does not migrate existing volumes to local storage. Ceph-backed workloads may stop working, and their data may become unavailable. Use this configuration only when creating or resetting a disposable debugging cluster.

## Verify the storage class

Wait for the application to become healthy, then confirm that Kubernetes exposes the `local-path` storage class:

```bash
kubectl get storageclass
```

The output marks `local-path` as `(default)` when `storageClass.defaultClass` is enabled. The provisioner creates a volume on the node where its consuming workload is scheduled.

## Use local storage

We can now use the `local-path` storage class instead of the default one.
For example, we can make the **Kubernetes as a Service** etcd datastores use the local disks:

```yaml
clusters:
  local:
    systemConfiguration:
      apps:
        kaas-datastore:
          values:
            dataStores:
              default:
                storageClassName: local-path
```

## Limitations

- **No redundancy:** the data has a single copy on one local disk.
- **No node failover:** if the node is unavailable, the workload cannot start on another node with the same data.
- **No disk-failure recovery:** replacing or losing the disk loses its local volumes.
- **Not shared storage:** local volumes are unsuitable for workloads that require storage accessible from multiple nodes.

Use the `local-path` storage class only for disposable debugging workloads. Use Ceph-backed storage classes for persistent or production workloads.
