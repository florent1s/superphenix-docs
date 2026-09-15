# Disk

A **Disk** is a persistent block storage volume provisioned within an Availability Zone (AZ). Disks exist independently of virtual machine instances, persisting data across VM restarts, reconfigurations, and deletions.

---

## Volume Ingestion Sources

When creating a disk, initial content can be populated from five sources:

| Source Type | Description | Typical Use |
| :--- | :--- | :--- |
| **`Blank`** | Allocates an unformatted block volume. | Secondary data drives, empty database volumes, or manual ISO installations. |
| **`HTTP / HTTPS`** | Downloads and extracts a raw or qcow2 image from a web URL. | OS boot disks from vendor cloud images (Ubuntu, Debian, Rocky Linux). |
| **`Registry (OCI)`** | Pulls a disk image packaged as an OCI container artifact. | Standardized corporate base images or immutable container disks. |
| **`Clone`** | Copies an existing disk within the same project and AZ. | Duplicating environments or preparing test copies of production disks. |
| **`Snapshot`** | Hydrates a volume from an existing VolumeSnapshot. | Restoring from a backup or rolling back to a previous checkpoint. |

!!! note "Image Import Sizing"
    When importing images via HTTP or OCI registry, the specified disk size must be larger than the uncompressed virtual size of the source image. Imports fail with a `DataVolume too small to contain image` error if the volume capacity is insufficient.

---

## Volume Operations

### Provisioning and Attachment

Disks can be provisioned as standalone resources under **Storage** > **Disks**, or created inline when launching an instance.

For a walkthrough on creating disks and attaching them during instance provisioning, see the [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#boot-disk) user guide.

- **AZ Scope**: Disks must reside in the same Availability Zone as the virtual machine mounting them. Cross-AZ mounting is not supported.
- **Bus Interface**: `virtio` is recommended for optimal I/O performance. `sata` and `scsi` are available for legacy operating systems or compatibility requirements.
- **Detachment**: To prevent filesystem corruption, shut down the instance before detaching a disk so that in-memory write caches are flushed cleanly.

### Online Resizing

Disks can be expanded online from the disk details page by selecting **Resize** and entering a larger capacity:

- The underlying storage volume expands immediately without downtime.
- Disks can only grow; shrinking an existing disk is not supported.
- Once the volume is resized in the platform, expand the partition and filesystem within the guest OS (e.g. using `growpart` and `resize2fs` on Linux).

### Deletion Protection

The platform prevents deleting disks that are currently attached to an instance or managed by platform services like Kubernetes-as-a-Service (KaaS). The volume must be detached or released before deletion is permitted.

---

## Configuration Reference

| Parameter | Description | Options |
| :--- | :--- | :--- |
| **Name** | Identifies the disk resource within the project. | Alphanumeric string and hyphens. |
| **Availability Zone** | Physical zone where the block volume is stored. | Must match the target VM's AZ. |
| **Source Type** | Mechanism used to initialize the disk. | `Blank`, `HTTP`, `Registry`, `Clone`, `Snapshot`. |
| **Storage Class** | Performance and replication profile in Ceph. | Platform storage pools (e.g., standard RBD, NVMe). |
| **Volume Mode** | Storage interface presentation. | `Filesystem` (default) or `Block`. |
| **Bus Interface** | Controller emulation for VM attachment. | `virtio`, `sata`, `scsi`, `auto`. |

---

## Disaster Recovery and Replication

For workloads requiring multi-site resilience, disks can be configured with asynchronous replication to a secondary Availability Zone or remote cluster:

- **Role Awareness**: The console indicates whether the volume is currently acting as `Primary` (read-write) or a secondary replica.
- **Sync Telemetry**: Real-time metrics display the last sync timestamp, synchronization duration, and volume of transferred changed blocks.
- **Lineage Tracking**: Disks created from disaster recovery plans display metadata indicating their origin.
