# Disk

A **Disk** is a high-performance, persistent virtual block storage device in Superphenix. Disks provide durable storage for virtual machine instances, functioning just like physical hard drives or SSDs.

Because Disks exist independently of virtual machines, your data remains completely safe even if you stop, rebuild, or delete an instance. You can attach a disk as a bootable root volume, attach secondary disks for databases and file storage, take point-in-time snapshots, and dynamically expand capacity as your needs grow.

---

## Key Features

- **Decoupled Persistence**: Data persists independently of VM lifecycles—attach, detach, and move disks between instances in the same Availability Zone.
- **Multiple Import Sources**: Create a blank drive, import an OS cloud image over HTTP/HTTPS, pull a container disk from an OCI registry, clone an existing disk, or restore from a snapshot.
- **Dynamic Online Resizing**: Expand disk size at any time without downtime or data loss.
- **High-Performance Storage Classes**: Choose storage profiles backed by fast NVMe pools or resilient, distributed Ceph RBD clusters.
- **Safety Guardrails**: Built-in protection prevents accidental deletion of mounted disks or disks managed by Kubernetes-as-a-Service (KaaS).
- **Disaster Recovery Telemetry**: Monitor continuous asynchronous cross-zone replication status, sync durations, and transferred data volumes in real time.

---

## Volume Ingestion Sources

When creating a disk, you can initialize its contents using one of five sources:

| Source Type | Description | When to Choose |
| :--- | :--- | :--- |
| **Blank** | Creates an empty, unformatted volume. | Secondary data storage, database drives, or manual OS installation from an ISO CD-ROM. |
| **HTTP / HTTPS** | Downloads and expands a disk image (raw or qcow2) directly from a remote web URL. | Boot disks created from vendor cloud images (e.g., Ubuntu, Debian, Rocky Linux). |
| **Registry (OCI)** | Streams a disk image packaged inside an OCI container image. | Immutable container disks or standardized corporate base images. |
| **Clone** | Creates a fast copy of another existing disk in the same project and AZ. | Duplicating environments or spinning up test copies of production data. |
| **Snapshot** | Hydrates a new disk directly from a previously captured VolumeSnapshot. | Restoring from a backup or rolling back to an earlier recovery checkpoint. |

!!! tip "Tip: Sizing for Image Imports"
    When importing an image via HTTP or OCI registry, make sure your disk size is larger than the *uncompressed* virtual size of the source image. If the disk is smaller, the import will fail with a `DataVolume too small to contain image` error.

---

## Volume Lifecycle & Management

### Creating and Attaching Disks

Disks can be provisioned as standalone volumes under **Storage** > **Disks**, or created inline while launching a virtual machine.

For a complete walkthrough on creating disks and attaching them during instance provisioning, see the **[Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#boot-disk)** user guide.

#### Attaching a Disk to an Instance
Disks in the same Availability Zone can be attached to any running or stopped instance via the instance details **Storage** tab. Select the volume and bus interface (`virtio` is recommended for high performance).

#### Detaching a Disk Safely
To protect against filesystem corruption, always detach disks cleanly:
1. Shut down the virtual machine gracefully via the web console.
2. Once the instance status is **Stopped**, open the **Storage** tab.
3. Select **Detach** from the disk options menu.
The detached disk remains available in your project and can be reattached to another instance.

---

### Expanding Disk Capacity

When your workload runs low on disk space, you can expand your volume dynamically from the disk details page by selecting **Resize** and specifying the new capacity.

The underlying storage pool expands the volume immediately without downtime. Once expanded, resize the partition and filesystem in the guest operating system (e.g., using `growpart` and `resize2fs` on Linux).

!!! note "Disks Can Only Grow"
    Block storage volumes can be expanded, but they cannot be shrunk. Always choose your target size carefully.

---

## Configuration Reference

| Setting | Description | Recommended Choice |
| :--- | :--- | :--- |
| **Name** | Identifies your disk across the project. | Descriptive name (e.g., `api-root-disk`, `redis-data`). |
| **Availability Zone** | Physical zone hosting the storage. | Must match the AZ of the virtual machine mounting it. |
| **Source Type** | How the disk is initialized. | `HTTP` for OS boot disks; `Blank` for data volumes. |
| **Storage Class** | Performance and replication profile. | Default Ceph RBD pool; NVMe/SSD for low-latency databases. |
| **Volume Mode** | `Filesystem` (standard) or `Block` (raw device). | `Filesystem` for almost all workloads. |
| **Bus Interface** | Controller emulation (`virtio`, `sata`, `scsi`). | `virtio` for maximum performance and throughput. |

---

## Disaster Recovery & Cross-Zone Replication

For mission-critical datasets, Superphenix supports automated asynchronous replication to a secondary Availability Zone or remote cluster:

- **Replication Status**: The console displays whether the disk is currently operating as `Primary` (active read/write) or a secondary replica.
- **Live Sync Telemetry**: Track real-time synchronization metrics:
  - Time since last synchronization.
  - Duration of the last sync run.
  - Total volume of data transferred (MB/GB).
  - Health status and replication alerts.
- **Disaster Recovery Lineage**: Disks created from disaster recovery plans display clear visual badges in the console indicating their recovery origin.

---

## Best Practices

- **Stop Instances Before Detaching**: Always power off a virtual machine before detaching persistent volumes to ensure in-flight write caches are flushed cleanly.
- **Standardize on VirtIO**: Use the `virtio` bus interface for all modern Linux and Windows instances to avoid hypervisor emulation bottlenecks.
- **Use Separate Disks for OS and Data**: Keep your operating system on a dedicated root disk and application data on separate data disks. This makes OS upgrades, restores, and volume resizing much simpler.
- **Pair with Snapshots**: Schedule regular automated snapshots for all disks containing critical application state.
