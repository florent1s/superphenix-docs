# Instance Snapshot

An **Instance Snapshot** captures the state of a virtual machine at a specific point in time, including its hardware configuration (vCPU, memory, network interfaces, firmware) and all attached persistent disks.

Unlike a standalone disk snapshot that only captures a single volume, an instance snapshot coordinates across every attached disk simultaneously to provide multi-disk crash or filesystem consistency.

---

## Snapshot Mechanics

Instance snapshots rely on underlying copy-on-write storage pointers and guest agent coordination:

1. **Guest Filesystem Freezing**: If `qemu-guest-agent` is running inside the VM, the platform issues an `fsfreeze` command to flush dirty in-memory buffers and file system journals to disk.
2. **Storage Snapshot Execution**: The underlying storage driver records block pointers across all attached volumes simultaneously.
3. **Thaw**: The guest filesystems are immediately unfrozen (`fsthaw`), allowing normal I/O operations to resume.
4. **Metadata Capture**: The virtual machine specification (CPU, RAM, device buses, network interfaces) is recorded alongside the child disk snapshots.

If `qemu-guest-agent` is not installed or responsive, the snapshot proceeds without `fsfreeze`, resulting in a crash-consistent rather than filesystem-consistent snapshot.

---

## Snapshot Operations and Restoration

### Creating a Snapshot

Snapshots can be created while the instance is running or stopped:

- From the console: Navigate to **Compute** > **Instances**, open **Options** (or the **Snapshots** tab), and select **Create Snapshot**.
- Via GitOps: Declare a snapshot resource targeting the virtual machine.

For details on navigating instance actions, see the [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#start-and-access-the-vm) user guide.

### Restoring from a Snapshot

Restoring an instance rolls back its virtual hardware and all attached disks to the exact state captured in the snapshot:

1. In the console, go to **Compute** > **Instances**, select the target instance, and open the **Snapshots** tab.
2. Select the restore point and confirm the restoration.

!!! warning "Restoration Overwrites Current Data"
    Restoring an instance overwrites the contents of all attached persistent disks with the snapshot data. Any data written since the snapshot was taken is permanently replaced.

After restoration completes, the instance powers on according to its configured Run Strategy.

---

## Child Disk Snapshots

Every instance snapshot automatically generates corresponding **Child Disk Snapshots** for each attached volume.

The snapshot details view displays:

- Source disk name and capacity.
- Creation timestamp.
- Readiness status.

Child disk snapshots can also be used independently to hydrate new standalone disks under **Storage** > **Disks**.

---

## Scheduled Snapshot Policies

To automate backups across instances:

- **Schedule Cadence**: Define cron expressions (e.g. daily at 02:00 UTC or weekly).
- **Retention Rules**: Set expiration periods (e.g. 7 days, 30 days) to prune older snapshots automatically.
- **Label Selectors**: Apply key-value labels to instances (e.g., `backup: daily`) so that matching workloads are automatically included in the backup schedule.
