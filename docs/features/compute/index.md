# Compute

Superphenix Compute provides on-demand, cloud-native virtual machines designed for high performance, reliability, and ease of management. Whether you are running microservices, relational databases, or legacy enterprise workloads, Compute offers dedicated CPU and memory, flexible networking, and persistent storage.

## Services

<div class="grid cards" markdown>

-   **[:material-server: Instances](instance.md)**

    ---

    Create and manage high-performance virtual machines with dedicated vCPUs, scalable RAM, multi-network attachments, and web-based consoles.

    [:octicons-arrow-right-24: Learn about Instances](instance.md)

-   **[:material-camera: Instance Snapshots](instance-snapshot.md)**

    ---

    Capture point-in-time, full-state snapshots of your virtual machines and all attached disks for instant rollbacks and safe upgrades.

    [:octicons-arrow-right-24: Learn about Instance Snapshots](instance-snapshot.md)

</div>

## Key Capabilities

- **Flexible Sizing & Performance**: Configure exact vCPU and RAM allocations to match your workload's requirements.
- **Isolated Multi-Interface Networking**: Connect instances to private project subnets with automated IP allocation or custom static IPs.
- **Persistent Block Storage**: Attach bootable root disks and high-speed secondary data disks powered by resilient distributed storage.
- **Interactive Consoles**: Access virtual machines directly from your browser using the built-in Serial Console or graphical VNC Console.
- **Enterprise Firmware & Security**: Deploy modern workloads with UEFI firmware, Secure Boot validation, and virtual TPM (vTPM 2.0).
- **Automated Bootstrapping**: Initialize instances seamlessly on first boot using cloud-init and managed SSH key injection.

---

## User Guides

For step-by-step console tutorials and practical examples:

- **[Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md)**: Step-by-step walkthrough of creating and booting an Ubuntu virtual machine with persistent storage and cloud-init.
- **[Expose a VM with an Elastic IP](../../user-guides/virtual-machines/expose-with-eip.md)**: Guide to provisioning an Elastic IP, configuring NAT, and connecting via SSH.
- **[Virtual machines user guide overview](../../user-guides/virtual-machines/index.md)**: Prerequisites, default images, and common workflow concepts.
