# Subnet

A **subnet** is an **L2 segment** with an IP range (**CIDR**) where you place instances. Think of it like a VLAN behind a router in a traditional design.

- A subnet **belongs to one VPC**. The **VPC** routes traffic to other subnets in the same VPC and toward the **Internet** when configured.
- **Custom gateway**: You can define how the subnet’s **default gateway** relates to the VPC router (advanced layouts).
- **Custom CIDR**: The subnet’s **CIDR block** is fully under your control within platform limits.
- **Exclude IPs from IPAM**: Reserve addresses (for gateways, load balancers, or static services) so **DHCP/IPAM** never hands them out.

## Create a subnet

1. Choose **IP family**: **IPv4**, **IPv6**, or **Dual stack** (both).
2. Set the **CIDR**: defines which IPs IPAM may assign to VMs.
3. Prefer **reasonably sized** prefixes (for example **`/24`** for IPv4) to avoid overlap and keep address plans manageable. Use **private** ranges appropriate to your environment (for example `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).

## IPAM

If you **do not** set a static IP on a VM, **IPAM** picks a free address **inside the subnet CIDR** (DHCP-style allocation). **Static** assignments bypass IPAM for that address.

## NAT gateway and “fully private” subnets

When creating a subnet you can:

- **Enable NAT gateway**: Instantiates a **NAT gateway** in the subnet so **outbound** traffic can reach the **Internet**. The subnet’s **routing table** is updated so traffic flows:

  `VM → default gateway → NAT gateway → Internet`

- **Fully private / isolated**: A stricter mode that **isolates** the subnet so it is **not reachable** even from other subnets in the same VPC. Use **firewall** rules for finer control when you only need to restrict access, not full isolation.

Subnets **host NAT gateways** when you enable them: the gateway is a resource in that subnet that provides **SNAT**, **DNAT**, and **1:1 NAT** (floating IP / **FIP**) patterns toward the Internet or peer networks.

After creation, the subnet detail shows **CIDR**, **gateway**, **VPC**, and (if enabled) the **NAT gateway** IP.