# Instance

An **Instance** is an on-demand virtual machine running on the Superphenix cloud platform. Built on modern, cloud-native virtualization technology, Instances provide dedicated virtual CPU, memory, persistent storage, and private networking—giving you complete operating-system-level control without the complexity of managing physical servers.

Whether you need to run high-throughput web applications, host production databases, or run enterprise software requiring custom kernel settings or Windows environments, Instances give you the flexibility and performance you need.

---

## Common Use Cases

- **Web Services & Application Backends**: Run high-concurrency APIs, microservices, and web applications with dedicated CPU and memory allocations.
- **Production Databases & Caches**: Deploy persistent databases (PostgreSQL, MySQL, Redis, Cassandra) backed by high-IOPS persistent storage.
- **Legacy & Specialized Environments**: Run workloads requiring custom Linux kernels, non-containerizable daemons, or Windows Server operating systems.
- **Kubernetes-as-a-Service (KaaS)**: Power upstream Kubernetes worker nodes managed automatically by the platform.
- **Development & CI/CD Runners**: Spin up ephemeral or long-lived virtual environments for automated builds and testing.

---

## Key Features

- **Flexible Compute Sizing**: Choose from 1 to 32 virtual CPU cores and 1 to 64 GB of RAM, tailored to your application's exact resource requirements.
- **Persistent Storage & Media**: Attach high-speed root and secondary block storage disks, live-mount ISO images for OS installation, or use pre-built container disk images.
- **Multi-Interface Networking**: Connect your instance to one or more private project subnets, with automatic IP assignment (IPAM) or custom static IPs.
- **Browser-Based Consoles**: Troubleshoot and manage your instance directly from your web browser using the text-based **Serial Console** or graphical **VNC Console** with an on-screen keyboard.
- **Automated Initialization (cloud-init)**: Automatically provision user accounts, inject SSH keys, set passwords, and run startup scripts on the first boot.
- **Enterprise Firmware & Security**: Toggle between legacy BIOS and UEFI firmware, enable Secure Boot for signed kernels, preserve boot settings with Persistent EFI, and utilize virtual TPM (vTPM 2.0).

---

## Creating and Managing Instances

Instances are provisioned, monitored, and managed directly from the Superphenix web console or declaratively via GitOps.

For step-by-step guidance on creating, configuring, and accessing virtual machines, refer to the dedicated user guides:

- **[Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md)**: Walk through the complete instance creation wizard—configuring vCPU/RAM, provisioning boot disks from cloud images, setting up cloud-init and SSH keys, and attaching subnets.
- **[Expose a VM with an Elastic IP](../../user-guides/virtual-machines/expose-with-eip.md)**: Assign a public IPv4 address (EIP) to your instance and establish remote SSH connectivity.
- **[Virtual machines user guide overview](../../user-guides/virtual-machines/index.md)**: Review prerequisites, default images, and cloud-init credentials.

---

## Configuration Reference

### Lifecycle & Run Strategies

The **Run Strategy** determines how Superphenix manages the power state of your virtual machine:

| Run Strategy | Description | Best For |
| :--- | :--- | :--- |
| **`Always`** *(Recommended)* | Keeps the instance running continuously. If the guest crashes or shuts down, the platform automatically restarts it. | Production services, web applications, databases. |
| **`Manual`** *(Default)* | Leaves power control entirely up to you. The instance starts only when you click **Start** and stays off when shut down. | Development, staging, testing, maintenance. |
| **`RerunOnFailure`** | Automatically restarts the instance only if it terminates unexpectedly or crashes. | Batch jobs, automated pipelines. |
| **`Once`** | Runs the instance once and never restarts it upon exit, even on crash. | One-off migration tasks, short diagnostics. |
| **`Halted`** | Guarantees the instance remains powered off until you change its strategy. | Archived or decommissioned instances. |

---

### Compute Sizing & CPU Topology

You can tailor virtual CPU and memory capacity to your workload needs:

- **vCPU Cores**: Allocate 1, 2, 4, 8, 16, or 32 virtual cores.
- **RAM**: Allocate 1, 2, 4, 8, 16, 32, or 64 GB of memory.
- **CPU Topology**: For specialized workloads, configure sockets, cores, and threads in the Advanced settings to optimize guest NUMA and threading behavior.

---

### Storage Disks & Media

Instances support multiple storage types and bus interfaces:

- **Root & Secondary Disks**: Attach persistent block storage volumes (Disks) created in the same AZ.
- **CD-ROM**: Attach an ISO image to boot rescue tools or perform manual OS installations.
- **Container Disks**: Mount immutable pre-packaged disk images directly from OCI container registries.

#### Storage Bus Options

| Bus Interface | Description | When to Use |
| :--- | :--- | :--- |
| **`virtio`** *(Recommended)* | High-performance paravirtualized driver. Provides maximum I/O throughput with minimal hypervisor overhead. | Modern Linux distributions and Windows instances with VirtIO drivers installed. |
| **`sata`** | Standard Serial ATA emulation. | Older operating systems or custom rescue media lacking VirtIO storage drivers. |
| **`scsi`** | Emulated SCSI controller. | Enterprise software requiring standard SCSI commands. |
| **`auto`** | Lets the platform choose the optimal bus based on OS preference. | Default selection for general workloads. |

---

### Networking

- **Subnet Attachment**: Connect your instance to one or multiple isolated project subnets. Each attached subnet becomes a separate virtual network interface (`Interface 0`, `Interface 1`, etc.).
- **IP Assignment**:
  - **Automatic (IPAM)**: The platform automatically leases an available IP address from the subnet's CIDR range.
  - **Static IP**: Enter an explicit IPv4 or IPv6 address within the subnet range.
- **Interface Model**: Select `virtio` for high-throughput, low-latency networking, or `e1000` for legacy operating systems.
- **Public Connectivity**: To make your instance accessible from the internet, associate an **Elastic IP (EIP)** with the instance's private IP. For a complete walkthrough, see the [Expose a VM with an Elastic IP](../../user-guides/virtual-machines/expose-with-eip.md) user guide.

---

### First-Boot Initialization (cloud-init) & SSH Keys

Superphenix simplifies instance setup using industry-standard **cloud-init**:

- **Default Cloud-Init**: Automatically provisions a default `spx` user account with passwordless `sudo` privileges, generates a secure initial password (visible on the instance details page), and configures DNS resolvers.
- **Custom Cloud-Init**: Provide your own `#cloud-config` YAML to install packages, run shell commands, write configuration files, or create additional user accounts.
- **SSH Key Injection**: Select public keys from your project's SSH Key catalog. These keys are automatically injected into the default user's `~/.ssh/authorized_keys` file for secure, passwordless authentication.

---

### Firmware & Security Options

In the **Advanced Options** section, you can configure enterprise-grade hardware security features:

- **Bootloader (BIOS vs. UEFI)**: Choose legacy BIOS for older operating systems or modern UEFI for modern operating systems and GPT partitioning.
- **Secure Boot**: Enforce UEFI Secure Boot verification so that only cryptographically signed kernels and bootloaders can execute.
- **Persistent EFI**: Retains NVRAM variables across instance reboots, preserving custom bootloader orders and EFI keys.
- **vTPM (Virtual TPM 2.0)**: Provides a virtualized Trusted Platform Module for cryptographic key storage, BitLocker encryption on Windows, and attestation.

---

## Connecting to Your Instance

### 1. SSH (Network Access)

Once your instance is running and has network connectivity, connect using your favorite terminal and your SSH key:

```bash
ssh spx@<instance-ip>
```

If you did not inject an SSH key, retrieve the generated one-time password from the **Storage** > **UserData** section of the instance details page. For detailed steps on locating your IP and testing SSH connectivity, see [Create a virtual machine](../../user-guides/virtual-machines/create-a-vm.md#start-and-access-the-vm) and [Expose a VM with an Elastic IP](../../user-guides/virtual-machines/expose-with-eip.md#test-the-connection).

### 2. Browser-Based Consoles

When an instance has no network connectivity, during early boot troubleshooting, or when recovering from a network misconfiguration, you can access the instance directly from your browser:

- **Web Serial Console**: Open the **Options** menu and click **Serial**. This opens an interactive text console connected to the virtual serial port.
- **Web VNC Console**: Open the **Options** menu and click **Console**. This opens a graphical desktop console complete with an on-screen virtual keyboard for sending key combinations such as `Ctrl+Alt+Delete`.

---

## Lifecycle Operations

From the instance details page, you can perform power and maintenance actions at any time:

- **Start**: Powers on a halted instance.
- **Shutdown (ACPI)**: Sends a graceful shutdown signal to the guest operating system, allowing services to flush data and exit cleanly.
- **Force Shutdown**: Immediately powers off the virtual machine (equivalent to pulling the power plug). Use this only when the guest OS is frozen.
- **Restart**: Reboots the operating system cleanly.
- **Create Snapshot**: Captures a full point-in-time snapshot of the instance and all its disks.

---

## Best Practices & Pro Tips

!!! tip "Install the QEMU Guest Agent"
    Always install and enable the `qemu-guest-agent` package inside your guest OS (`sudo apt install qemu-guest-agent` on Ubuntu/Debian). This agent allows the platform to accurately display assigned IP addresses, monitor memory metrics, and freeze filesystems cleanly during snapshots.

!!! tip "Use VirtIO Drivers for Maximum Performance"
    Modern Linux distributions include VirtIO drivers out of the box. If you run Windows, install the VirtIO guest drivers to ensure optimal disk and network throughput.

!!! note "Production High Availability"
    Use the `Always` run strategy for production workloads so that your virtual machines automatically recover in the event of underlying host maintenance or system reboots.

!!! warning "Detaching Disks Safely"
    To prevent data corruption, always stop your instance before detaching persistent disks.
