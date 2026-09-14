# EIP (Elastic IP)

**Elastic IPs** map **public** addresses to **private** addresses in your subnet (NAT / DNAT patterns).

**Prerequisites**

- Enable a **NAT gateway** on the **subnet** where you expose workloads. That NAT gateway handles traffic toward your private instances.

## Create and attach an EIP

1. Open the **subnet** where you need Internet access or inbound publishing.
2. Use **Attach EIP** (or equivalent in **Options**).
3. Name the EIP.

Then choose how NAT behaves:

| Mode              | Use case                                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Internal IP**   | **1:1 NAT** between one **private** VM IP and the EIP. Inbound to the EIP hits that VM; outbound from that VM uses the EIP on the Internet. |
| **Internal CIDR** | **SNAT** a whole **CIDR** inside the subnet (often the **full subnet CIDR**) so outbound connections use the EIP as source.                 |

**DNAT (port forwarding)**: Map **external** ports on the EIP to an **internal** IP and **port** on a VM inside the CIDR (open a service to the Internet).

!!! note "One NAT gateway per subnet (today)"

    A subnet’s **VPC NAT gateway** currently serves **that subnet’s** outbound path. If a subnet needs Internet access or inbound reachability, it needs **its own** NAT gateway and **its own** EIPs. **Sharing** one NAT gateway across multiple subnets may come in a future release to save public IPs.

Created EIPs appear under **Network → EIPs** (or equivalent).
