# Snapshot

A **Snapshot** is an instantaneous, point-in-time copy of a persistent disk. Because snapshots use copy-on-write technology, they are created in seconds and only consume additional storage space as new data is written to the source disk.

Snapshots provide the easiest and safest way to protect your data before making changes, recover from accidental deletions, or clone production data into isolated test environments.

---

## Why Use Snapshots?

- **Pre-Maintenance Safety Net**: Capture an immediate snapshot before performing system updates, database migrations, or configuration edits. If anything goes wrong, you can roll back with confidence.
- **Instantaneous Creation**: Snapshots complete almost instantly without pausing or interrupting your running applications.
- **Hydrate New Disks**: Turn any snapshot into a brand-new, independent persistent disk—ideal for staging environments, data analytics, or forensics.
- **Resize on Restore**: When restoring a snapshot to a new disk, you can optionally provision a larger capacity to scale storage seamlessly.
- **Automated Protection**: Set up recurring snapshot schedules with retention limits so your critical data is protected automatically.

---

## How It Works

Superphenix utilizes underlying storage copy-on-write mechanisms:

1. **Instant Freeze of Metadata**: When a snapshot is requested, the system freezes the block allocation map at that exact moment.
2. **Delta Tracking**: The original data blocks remain unchanged. Any future writes made by your workload are directed to new blocks.
3. **Space Efficiency**: A snapshot initially consumes virtually zero extra disk space; it only grows as blocks on the active volume change over time.

---

## Snapshot Lifecycle & Restoration

### Creating a Snapshot

Snapshots can be created on-demand for any disk in your project via **Storage** > **Disks** by selecting **Create Snapshot** on the target disk. The snapshot is created almost instantly using copy-on-write pointers and transitions to **Ready to use**.

---

### Restoring a Snapshot to a New Disk

Restoring a snapshot hydrates a brand-new, independent persistent disk containing all data from the restore point:

- Navigate to **Storage** > **Snapshots** and select **Restore to Disk** (or choose *Snapshot* as the source when creating a new disk).
- Configure the new disk name, target Availability Zone, and desired capacity (which can be equal to or larger than the original disk).
- Once hydrated, the new volume can be attached to any virtual machine in the same AZ. For guidance on attaching disks to instances, see the [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#boot-disk) user guide.

---

## Automated Snapshot Schedules & Policies

To ensure business continuity without manual intervention, enroll your disks into scheduled backup policies:

- **Schedule Cadence**: Define cron expressions (e.g., daily at midnight or weekly on Sundays).
- **Retention Rules**: Set expiration periods (e.g., retain daily snapshots for 7 days, weekly snapshots for 4 weeks) so expired snapshots are deleted automatically.
- **Targeting by Labels**: Apply custom labels (such as `environment: production`) to automatically include newly provisioned disks in backup policies.

---

## Best Practices

- **Quiesce Active Databases**: While block snapshots are crash-consistent, flushing pending transactions or putting your database into backup mode (e.g., `pg_start_backup()` for PostgreSQL) before triggering a snapshot ensures full application-level consistency.
- **Enforce Retention Limits**: Always specify an expiration time for recurring snapshots to avoid accumulating stale data that consumes storage quota.
- **Regularly Test Restores**: Test your recovery process periodically by hydrating a snapshot to a test disk and mounting it to a staging instance.
