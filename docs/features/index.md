# Features

Superphenix is organized in two layers: the **foundation** provides the infrastructure platform (IaaS and operations), and **managed services** consume it to deliver higher-level offerings. The **web console** exposes these capabilities as **products** (compute, storage, network, SSH keys, and more), scoped by **organization**, **project**, and **availability zone (AZ)**.

Read **[Tenancy and console](tenancy-and-console.md)** first for organizations, projects, IAM, and how to navigate resources.

## Foundation

The foundation provides the core capabilities that the rest of the platform relies on.

<div class="grid cards" markdown>

-   :lucide-server:{ .lg .middle } __Virtualization__

    ---

    **Instances (VMs)**, **live migration**, **snapshots** (VM and scheduled volume), **restores**, **cloud-init**, SSH, serial, and VNC.

    [:octicons-arrow-right-24: Virtualization](virtualization.md)

-   :lucide-network:{ .lg .middle } __Network__

    ---

    **VPCs**, **subnets**, **NAT gateways**, **load balancers**, **Elastic IPs (EIPs)**, and **firewalls**: CSP-style networking with IPAM and optional Internet exposure.

    [:octicons-arrow-right-24: Network](network.md)

-   :lucide-hard-drive:{ .lg .middle } __Storage__

    ---

    **Disks** and **disk snapshots**: block volumes (and classes such as high-speed), create from **blank**, **HTTP image**, **snapshot**, or **clone**; online resize.

    [:octicons-arrow-right-24: Storage](storage.md)

</div>

## How they fit together

- **Tenancy**: Organizations and projects isolate resources and access; see [Tenancy and console](tenancy-and-console.md).
- **Foundation**: Virtualization runs VMs; the network layer provides VPCs and subnets; storage backs disks; tooling drives GitOps and the console.
- **Managed services**: When enabled, PaaS and SaaS consume the same virtualization, storage, and network primitives.

See [Architecture overview](../architecture/index.md) for AZs, regions, and central administration.
