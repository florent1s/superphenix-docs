# Backup & Disaster Recovery

Superphenix Backup and Disaster Recovery (DR) provides automated volume snapshot scheduling and asynchronous cross-zone block replication for persistent disks.

---

## Protection Strategies

| Strategy                     | Mechanism                                                               | RPO           | RTO                         | Primary Use Case                                              |
| :--------------------------- | :---------------------------------------------------------------------- | :------------ | :-------------------------- | :------------------------------------------------------------ |
| **Scheduled Snapshots**      | Point-in-time copy-on-write snapshots created on a schedule.            | Hours to days | Minutes (volume hydration)  | Periodic backups, compliance archives, development baselines. |
| **Asynchronous Replication** | Continuous delta block replication to a secondary AZ or remote cluster. | Minutes       | Minutes (replica promotion) | High-availability failover across zones, disaster recovery.   |

---

## Automated Snapshot Policies

Backup policies automate snapshot execution and retention across project disks:

- **Schedule Cadence**: Configured using cron expressions (e.g., daily at 01:00 UTC or weekly).
- **Retention Rules**: Set expiration periods (e.g., retain daily snapshots for 14 days) to automatically delete older recovery points.
- **Label Selectors**: Policies target disks matching specific key-value labels (e.g., `backup-policy: gold`). Newly created disks matching the label are automatically included in the schedule.

---

## Asynchronous Cross-Zone Replication

For workloads requiring resilience against zone-level failures, disks can be replicated asynchronously:

- **Roles**: Volumes operate as either `Primary` (active read-write volume attached to a VM) or `Secondary` (synchronized read-only replica in the target AZ).
- **Sync Telemetry**: The web console displays:
  - Time of last completed synchronization.
  - Duration of the last sync run.
  - Volume of changed blocks transferred (MB/GB).
  - Health status and replication alerts.
