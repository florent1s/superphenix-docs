# Storage

Superphenix storage services provide persistent block storage for virtual machines, volume snapshots, automated backup and cross-zone replication, and S3-compatible object storage.

## Storage Services

<div class="grid cards" markdown>

-   **[:material-harddisk: Disks](disk.md)**

    ---

    Persistent block storage volumes for virtual machines. Supports raw volumes, image imports, and online resizing.

    [:octicons-arrow-right-24: Disks](disk.md)

-   **[:material-camera-outline: Snapshots](snapshot.md)**

    ---

    Point-in-time, copy-on-write volume snapshots for recovery checkpoints and cloning new disks.

    [:octicons-arrow-right-24: Snapshots](snapshot.md)

-   **[:material-cloud-sync: Backup & DR](baas.md)**

    ---

    Automated snapshot scheduling policies and asynchronous cross-zone volume replication.

    [:octicons-arrow-right-24: Backup & DR](baas.md)

-   **[:material-bucket-outline: Object Storage](bucket.md)**

    ---

    S3-compatible object storage for application assets, logs, backups, and media files.

    [:octicons-arrow-right-24: Object Storage](bucket.md)

</div>

---

## Storage Types

| Service | Storage Type | Access Protocol | Scope & Use Case |
| :--- | :--- | :--- | :--- |
| **[Disks](disk.md)** | Block storage | Virtual disk bus (`virtio`, `sata`, `scsi`) | VM root boot disks, persistent database storage, local filesystems. |
| **[Snapshots](snapshot.md)** | Point-in-time copy | Storage volume restore | Recovery points, disk cloning, pre-maintenance backups. |
| **[Backup & DR](baas.md)** | Scheduled snapshot & replication | Cron policies & block replication | Retention compliance, offsite copies, cross-zone failover. |
| **[Object Storage](bucket.md)** | Object storage | S3 API over HTTPS | Static assets, media, logs, data archives, application file uploads. |
