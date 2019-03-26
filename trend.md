# {{ page.title }}

## Virtual Networks All the Way Down

For almost as long as there have been packet-switched networks, there
have been ideas about how to virtualize them, starting with virtual
circuits. But what exactly does it mean to virtualize the network?
 
Virtual memory is a helpful example. Virtual memory creates an
abstraction of a large and private pool of memory, even though the
underlying physical memory may be shared by many applications
and considerably smaller that the apparent pool of virtual
memory. This abstraction enables programmers to operate under the
illusion that there is plenty of memory and that no-one else is using
it, while under the covers the memory management system takes care of
things like mapping the virtual memory to physical resources and
avoiding conflict between users.
 
Similarly, server virtualization presents the abstraction of a virtual
machine (VM), which has all the features of a physical machine. Again,
there may be many VMs supported on a single physical server, and the
operating system and users on the virtual machine are happily unaware
that the VM is being mapped onto physical resources.
 
A key point is the virtualization of computing resources preserves the
abstractions and interfaces that existed before they were virtualized.
This is important because it means that users of those abstractions
don't need to change—they see a faithful reproduction of the resource
being virtualized. So what happens when we try to virtualize networks?
 
VPNs were one early success for virtual networking. They allowed
carriers to present corporate customers with the illusion that they
had their own private network, even though in reality they were
sharing underlying links and switches with many other users.  VPNs,
however, only virtualize a few resources, notably addressing and
routing tables. Network virtualization as commonly understood today
goes further, virtualizing every aspect of networking. That means that
a virtual network today supports all the basic abstractions of a
physical network—switching, routing, firewalling, load
balancing—virtualizing the entire network stack from layers two
through seven. In this sense, they are analogous to the virtual
machine, with its support of all the abstractions of a server: CPU,
storage, I/O, and so on.
 
To this end, *Virtual LANs* (VLANs) are how we typically virtualize an
L2 network. Supporting VLANs required a fairly simple extension to the
original 802.1 header specification, inserting a 12-bit VLAN ID
(`VID`) field between the `SrcAddr` and `Type` fields, as shown in
[Figure 1](#vlan). (This VID is typically referred to as a *VLAN tag*.)
There are actually 32-bits inserted in the middle of the header, but
the first 16-bits are used to preserve backwards compatibility with
the original specification (they use `Type = 0x8100` to indicate that
this frame includes the VLAN extension); the other four other bits
hold control information used to prioritizing frames. This means it is
possible to map 2$$^{12}$$ = 4096 virtual networks onto a single
physical LAN.

<figure class="line">
	<a id="vlan"></a>
	<img src="figures/impl/Slide4.png" width="500px"/>
	<figcaption>802.1Q VLAN tag embedded within an Ethernet (802.1) 
	header.</figcaption>
</figure>
 
VLANs proved to be quite useful to enterprises that wanted to isolate
different internal groups (e.g., departments, labs), giving each of
them the appearance of having their own private LAN. VLANs were also seen
as a promising way to virtualize L2 networks in cloud datacenters,
making it possible to give each tenant their own L2 network so as to
isolate their traffic from the traffic of all other tenants. But there
was a problem: the 4096 possible VLANs was not sufficient to account
for all the tenants that a cloud might host, and to complicate
matters, in a cloud the network needs to connect *virtual machines*
rather than the physical machines that those VMs run on.
 
To address this problem, another standard called *Virtual Extensible
LAN* (VXLAN) was introduced. Unlike the original approach, which
effectively encapsulated a virtualized ethernet frame inside another
ethernet frame, VXLAN encapsulates a virtual ethernet frame inside a
UDP packet. This means a VXLAN-based virtual network (which is often
referred to as an *overlay network*) runs on top of an IP-based
network, which in turn runs on an underlying ethernet (or perhaps in
just one VLAN of said underlying ethernet). VXLAN also makes it
possible for one cloud tenant to have multiple VLANs of their own,
which allows them to segregate their own internal traffic. This means
it is ultimately possible to have a VLAN encapsulated in a VXLAN overlay
encapsulated in a VLAN.

The powerful thing about virtualization is that when done
right, it should be possible to nest one virtualized resource inside
another virtualized resource, since after all, a virtual resource
should behave just like a physical resources and we know how to
virtualize physical resources! Or said another way, being able to
virtualize a virtual resource is the best proof that you have done a
good job of virtualizing the original physical resource.
 
The actual VXLAN header is simple. It includes a 24-bit *Virtual
Network Id* (VNI), plus some flag bits. It also implies a particular
setting of the UDP source and destination port fields (see
Section 5.1), with the destination port 4789 officially reserved for
VXLANs. Figuring out how to assign identifiers to virtual LANs (VLAN
tags) and virtual networks (VXLAN VIDs) is the easy part. This is
because encapsulation is the fundamental cornerstone of
virtualization; all you need to add is an identifier that tells you
which of many possible users this encapsulated packet belongs to.

The hard part is grappling with the idea of virtual networks being
nested (encapsulated) inside virtual networks, which is networking’s
version of recursion. The other challenge is understanding how to
automate the creation, management, and deletion of virtual networks,
and on this front there is still a lot of room for improvement.
Mastering this problem will be at the heart of networking in the next
decade.

> [!NOTE|label:Broader Perspective]
> To continue learning about the cloudification of the Internet, see
> [The Cloud is Eating the Internet](../scaling/trend.md)
