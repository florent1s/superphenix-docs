# Security Group

Firewall rules are expressed as **network policies** (Kubernetes-style semantics):

- **Default deny**: Traffic that is **not explicitly allowed** is **dropped** (allow-only model).
- **Match criteria**: Rules can target **CIDRs** and/or **label selectors** (for VMs or workloads).
- **Protocols**: **TCP**, **UDP**, and **SCTP** where supported.

The console UX may still evolve; power users can mirror the same rules in **GitOps** for review and audit.

## Create a Security Group