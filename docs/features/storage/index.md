# Storage

Superphenix provides enterprise-grade, highly available storage services designed for modern cloud workloads. From high-IOPS persistent block storage for virtual machines to scalable S3-compatible object storage for media and backups, you can store, protect, and manage your data with ease.

## Storage Services

<div class="grid cards" markdown>

-   **[:material-harddisk: Disks](disk.md)**

    ---

    Persistent block storage volumes for virtual machines. Provision empty disks, import OS images, and resize dynamically on demand.

    [:octicons-arrow-right-24: Learn about Disks](disk.md)

-   **[:material-camera-outline: Snapshots](snapshot.md)**

    ---

    Point-in-time, copy-on-write backups of individual disk volumes for instant recovery and hydrating new disks.

    [:octicons-arrow-right-24: Learn about Snapshots](snapshot.md)

-   **[:material-cloud-sync: Backup & DR](baas.md)**

    ---

    Automated backup schedules and continuous cross-zone replication to ensure business continuity and minimize RPO/RTO.

    [:octicons-arrow-right-24: Learn about Backup & DR](baas.md)

-   **[:material-bucket-outline: Object Storage](bucket.md)**

    ---

    S3-compatible, durable cloud object storage for media files, logs, database dumps, and application assets.

    [:octicons-arrow-right-24: Learn about Object Storage](bucket.md)

</div>

---

## Choosing the Right Storage

| Service | Storage Type | Protocol / Access | Typical Use Cases |
| :--- | :--- | :--- | :--- |
| **[Disks](disk.md)** | Block Storage | Mounted as virtual disk (`virtio`, `sata`) | OS boot disks, database storage, file systems, low-latency applications. |
| **[Snapshots](snapshot.md)** | Point-in-time copy | Internal storage restore | Pre-upgrade checkpoints, volume cloning, disaster recovery baselines. |
| **[Backup & DR](baas.md)** | Data Protection | Scheduled policies & replication | Compliance backups, offsite data copies, cross-zone disaster recovery. |
| **[Object Storage](bucket.md)** | Object Storage | AWS S3 REST API (HTTPS) | Web static assets, video/image media, raw data lakes, application backups. |
