# Backup & Disaster Recovery

Superphenix provides comprehensive **Backup** and **Disaster Recovery (DR)** capabilities to ensure your business data is resilient against hardware failures, accidental deletion, ransomware, and datacenter outages.

By combining **automated snapshot schedules** with **continuous asynchronous replication**, Superphenix lets you protect your workloads according to your exact business continuity goals without complex manual scripting.

---

## Core Concepts

Understanding two industry concepts helps you select the best data protection strategy:

- **Recovery Point Objective (RPO)**: The maximum acceptable age of data that can be lost when an unexpected disaster strikes. For example, a 15-minute RPO means you could lose at most 15 minutes of newly written data.
- **Recovery Time Objective (RTO)**: The maximum acceptable downtime to restore systems and resume business operations after an outage.

| Protection Strategy | Typical RPO | Typical RTO | Best Used For |
| :--- | :--- | :--- | :--- |
| **Continuous Replication** | Minutes | Minutes | Mission-critical databases, primary API backends, core transactional workloads. |
| **Scheduled Backups** | Hours / Daily | 15–30 Minutes | Standard application data, development environments, long-term compliance retention. |

---

## Key Capabilities

### 1. Automated Backup Schedules
- **Scheduled Snapshot Execution**: Automatically trigger point-in-time snapshots based on predefined cron schedules (e.g., hourly, daily, weekly).
- **Automated Retention Management**: Specify retention rules (such as *keep daily backups for 14 days, weekly backups for 8 weeks*) so the system safely prunes expired snapshots without manual intervention.
- **Dynamic Tag-Based Protection**: Assign labels to your disks (e.g., `backup-tier: gold`). Any new disks with this label are automatically enrolled into the matching backup schedule.

### 2. Cross-Zone Asynchronous Replication
- **Multi-Zone Redundancy**: Asynchronously replicate block storage volumes from a primary Availability Zone to a secondary target AZ or remote cluster.
- **Primary vs. Secondary Awareness**: Clear role tracking in the console showing which volume is the active primary and which is the synchronized replica.
- **Real-Time Replication Telemetry**:
  - **Last Sync Timestamp**: See exactly when the last replication completed (displayed in your local timezone with UTC details).
  - **Sync Duration & Data Volume**: Monitor how long each sync cycle takes and how many megabytes or gigabytes of changed blocks were transferred.
  - **Health Alerts**: Immediate status indicators if synchronization fails or experiences high latency.
- **DR Lineage Tracking**: Disks hydrated from disaster recovery plans automatically display badges and metadata identifying their recovery lineage.

---

## Managing Backup Policies

Backup policies automate snapshot execution and lifecycle management across project disks:

- **Policy Cadence**: Select predefined frequencies (such as daily at 01:00 UTC) or specify custom cron expressions.
- **Retention Rules**: Set expiration periods (e.g., 30 days) to automatically prune older recovery points and reclaim capacity.
- **Dynamic Label Selectors**: Match volumes using labels (e.g., `backup: enabled`) so that newly provisioned disks are immediately protected without updating policy definitions.

---

## Disaster Recovery & Failover Workflow

In the event of an availability zone incident or planned maintenance:

1. **Verify Replica Status**: Check the replication telemetry for your target volume to confirm the last successful sync timestamp.
2. **Promote the Replica**: In the disaster recovery management console, promote the secondary volume to become a standalone, active primary disk.
3. **Attach to Recovery Instance**: Launch or attach the promoted disk to a virtual machine instance running in the secondary Availability Zone.
4. **Update Network Routing**: Reassign your Elastic IP (EIP) or update your load balancer target group to direct traffic to the recovery instance.

---

## Best Practices

- **Tier Your Datasets**: Apply high-frequency cross-zone replication to mission-critical transactional databases, while using daily scheduled snapshots for less volatile application volumes.
- **Monitor Telemetry Trends**: Keep an eye on replication sync durations and data transfer volumes. A sudden increase in sync duration may indicate a surge in write activity or network bandwidth constraints.
- **Run Regular DR Drills**: Don't wait for an outage to test your recovery plan. Regularly practice promoting replicas and attaching them to test instances to verify data integrity and team readiness.
