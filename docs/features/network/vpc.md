# VPC (Virtual Private Cloud)

A **VPC** is an **L3 routing domain** that **contains subnets**. Subnets inside the **same** VPC can reach each other according to routing and firewall rules, similar to placing multiple VLANs behind one router that interconnects them.

- **L3 routing**: The VPC owns **route tables** and can host **custom static routes** to steer traffic between subnets, toward NAT, or to other next hops as you define.
- **Isolation**: Two subnets in **different** VPCs **cannot** talk privately; communication would go via the **Internet** (unless you add future **VPC peering** when available).

## Create a VPC