# Snapshot

A **Snapshot** is a point-in-time copy of a persistent disk. Snapshots are implemented using copy-on-write mechanisms in the underlying storage pool, creating immediate recovery points without copying all data blocks upfront.

---

## Mechanics and Consistency

- **Copy-on-Write**: Creating a snapshot records the block allocation table at that instant. Subsequent writes to the source volume allocate new blocks, preserving the original blocks for the snapshot.
- **Crash Consistency**: Standalone disk snapshots capture raw blocks as they exist on the storage layer. For multi-disk consistency across an entire VM, use [Instance Snapshots](../compute/instance-snapshot.md).

---

## Operations and Restoration

### Creating a Snapshot

Snapshots can be created on-demand for any disk in the project:

- In the console, go to **Storage** > **Disks**, locate the volume, and select **Create Snapshot**.
- Snapshots can also be triggered declaratively through GitOps or automated policies.

### Hydrating a New Disk from a Snapshot

Snapshots do not overwrite the source volume in-place. Instead, restoring a snapshot provisions a new persistent disk containing the captured data:

1. Under **Storage** > **Snapshots**, select **Restore to Disk** (or create a new disk with `Snapshot` as the source type).
2. Configure the target Availability Zone and disk size.
3. The new disk size may be equal to or larger than the source snapshot, but cannot be smaller.

Once the disk status becomes **Ready**, it can be attached to any instance in the same AZ. For attachment instructions, see the [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#boot-disk) user guide.

---

## Automated Snapshot Policies

Disks can be enrolled in automated snapshot policies:

- **Cron Schedules**: Set recurring execution intervals (e.g., hourly, daily at 00:00 UTC, or weekly).
- **Retention Rules**: Define expiration windows (e.g., retain daily snapshots for 14 days) to automatically purge expired recovery points.
- **Label Selectors**: Use key-value labels (e.g., `backup: enabled`) to automatically include matching disks in the policy.
