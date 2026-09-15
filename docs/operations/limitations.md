# Limitations

## NAT gateway traffic to its host node

Traffic leaving a NAT gateway is blackholed when its destination is an address
on the physical node that hosts the gateway. This behavior is caused by a
macvlan limitation: a macvlan interface cannot communicate directly with its
host.

As a result, a VM that uses the NAT gateway might be unable to reach a node
address. This can affect the [Getting started](../installation/getting-started.md)
deployment when the console DNS record points directly to a node. It is
especially problematic for Kubernetes as a Service clusters that need to reach
the Superphenix control plane.

Use a load balancer in front of the nodes and point the DNS record to the load
balancer. The load balancer forwards traffic to a node without relying on
direct communication between the NAT gateway and its host.

As a temporary workaround, schedule the NAT gateway on a node other than the
destination node. This is less reliable because rescheduling the gateway onto
the destination node causes the limitation to recur.
