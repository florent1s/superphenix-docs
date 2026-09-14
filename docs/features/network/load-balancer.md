# Load Balancer

A Superphenix load balancer is an **L3/L4** balancer. It exposes a **custom VIP** (virtual IP) that forwards to backends using **TCP**, **UDP**, or **SCTP** (protocol availability may vary by release). Listeners can reference **named ports** for clarity in larger configs.

## Virtual IP range

- The front **virtual IP** is chosen by you from **`198.18.0.0/16`** (IPv4 only at the time of writing).
- **Do not** reuse the same virtual IP on two load balancers.

A load balancer is tied to a **subnet**; routing inside the **VPC** lets **any VM in that VPC** reach the VIP when the VPC routes to the subnet that owns it. The **subnet** is inferred from **load balancer mapping** (see below).

## Mapping (backends)

**Mapping** defines where traffic is sent:

| Mode               | Behavior                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------- |
| **IP backends**    | Target backends by **raw IP** (static list).                                                  |
| **Label backends** | Target VMs matching a **label selector**; backends update **dynamically** when new VMs match. |

You then define **ports** (for example **80** / **443**) for forwarding.

**Health checks** probe backends; failing members are removed. Health check **source** addresses come from the **subnet** attached to the load balancer.

!!! warning "Same subnet for all backends"

    Selector or endpoint configuration must resolve to backends on a **single subnet**. Only the **first network interface** of each VM participates in load balancing: **secondary NICs** are not used. **All VMs behind one load balancer should sit in the same subnet.**

## Create a Load Balancer