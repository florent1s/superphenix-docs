# Instance Snapshot

An **Instance Snapshot** creates a point-in-time backup of an entire virtual machine, capturing its exact hardware configuration alongside all attached persistent disks. 

Unlike a standard disk snapshot that only backs up a single isolated volume, an instance snapshot coordinates across every disk attached to the VM simultaneously. This ensures multi-disk data consistency and allows you to instantly roll back your entire system if something goes wrong.

---

## Why Use Instance Snapshots?

- **Pre-Maintenance Checkpoints**: Take a snapshot right before updating operating system packages, upgrading databases, modifying network configs, or deploying major application releases.
- **Instant Rollback**: If an upgrade fails or data is accidentally corrupted, restore the instance to its exact previous state in minutes.
- **Zero-Downtime Live Backups**: Snapshots are taken while the instance is running, without requiring you to shut down or restart your services.
- **Coordinated Multi-Disk Consistency**: When your workload uses multiple disks (such as a root OS disk and dedicated database data disks), an instance snapshot freezes all disks at the same exact moment.
- **Automated Protection**: Enroll critical instances into recurring backup schedules with automated retention rules.

---

## How It Works

Instance snapshots leverage copy-on-write storage technology and guest-level coordination:

1. **Guest Filesystem Coordination**: If the `qemu-guest-agent` is running inside your virtual machine, the platform issues an `fsfreeze` command. This temporarily flushes all pending in-memory write buffers and journal transactions to disk.
2. **Atomic Storage Capture**: The underlying storage layer instantly records block pointers for each attached volume.
3. **Thaw & Resume**: The guest filesystem unfreezes immediately (typically in milliseconds). The VM continues running normally without any noticeable interruption.
4. **Metadata Preservation**: The snapshot also records the instance's CPU, memory, network interfaces, and firmware configurations.

!!! tip "Ensure Consistent Backups with Guest Agent"
    Always install the QEMU guest agent (`sudo apt install qemu-guest-agent`) on your virtual machines. This enables the platform to flush disk caches cleanly, guaranteeing file system integrity rather than just crash consistency.

---

## Snapshot Operations & Lifecycle

### Taking a Snapshot

Instance snapshots can be triggered at any time from the web console (under **Compute** > **Instances** > **Options** > **Create Snapshot**) or via GitOps automation.

During snapshot creation:

- The platform signals the guest agent inside the VM to flush pending transactions and freeze filesystems (`fsfreeze`).
- Underlying storage snapshots are coordinated across all attached volumes simultaneously.
- Virtual machine specifications (CPU, memory, networking, firmware) are captured alongside the child disk snapshots.

For details on managing running virtual machines and accessing instance actions, see the [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#start-and-access-the-vm) user guide.

---

### Restoring an Instance from a Snapshot

Restoring an instance rolls back its virtual hardware and all attached disks to the exact state captured in the snapshot:

1. In the console, navigate to **Compute** > **Instances**, select your instance, and open the **Snapshots** tab.
2. Select the desired restore point and confirm the restoration action.

!!! warning "Restoration Overwrites Current Data"
    Restoring a snapshot overwrites the current contents of the attached disks with the data from the snapshot. Any data written since the snapshot was taken will be lost. Back up any uncommitted data before proceeding.

Upon confirmation, the instance powers down, reverts its disks and configuration to the snapshot checkpoint, and restarts according to its configured Run Strategy.

---

## Inspecting Child Disk Snapshots

When you view an instance snapshot in the console, you can expand its details to see the individual **Child Disk Snapshots**. 

Each attached volume has its own corresponding snapshot record showing:
- Source disk name and capacity.
- Snapshot creation timestamp.
- Current readiness state.

This makes it easy to verify that all attached storage volumes were captured successfully during the snapshot operation.

---

## Automated Snapshot Schedules

Rather than taking manual snapshots before every change, you can configure automated snapshot schedules:

- **Schedule Frequency**: Define cron-based schedules (e.g., every night at 02:00 UTC, or weekly).
- **Retention Policies**: Automatically expire and delete older snapshots after a set retention period (e.g., retain for 7 days or 30 days) to prevent storage bloat.
- **Dynamic Label Targeting**: Tag instances with labels (e.g., `backup-tier: gold`) so that new workloads are automatically enrolled into matching backup schedules.

---

## Best Practices

- **Quiesce High-Throughput Databases**: For heavy transactional databases (PostgreSQL, MySQL, Oracle), run a brief database-level checkpoint or flush command before taking snapshots to ensure application-level consistency.
- **Set Expiration Windows**: Always pair scheduled snapshots with retention policies so that outdated snapshots are automatically pruned.
- **Test Your Restores**: Periodically perform disaster recovery drills by restoring a snapshot into a staging instance to verify backup health and recovery time.
