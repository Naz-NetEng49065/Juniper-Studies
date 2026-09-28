# Introduction to MPLS

**Routing foundations, the motivations for MPLS, and the key terms you need before going further**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals · Part 01

[Index](../README.md) · [Next: The Mechanics of MPLS](02-mpls-mechanics.md) →

**Contents:** [1 Foundational Routing Concepts](#module-1-foundational-routing-concepts) · [2 Why MPLS](#module-2-the-why-of-mpls-motivations) · [3 Next Steps](#module-3-next-steps-in-your-mpls-journey) · [4 Glossary](#module-4-glossary-of-key-terms)

---

## Module 1: Foundational Routing Concepts

**Objective:** review how routers forward IP traffic, the separate roles of the routing and forwarding tables, and how BGP next hops are resolved.

### 1.1 Routing vs. forwarding tables

To understand MPLS you need to separate the two main tables a Junos router uses: the **routing table** and the **forwarding table**.

The **routing table** (Routing Information Base, RIB) is the router's control-plane database. It holds every prefix learned from every configured routing protocol (BGP, OSPF, IS-IS), plus static and directly connected routes. The router's job is to analyse this full list and select the single best path for each destination prefix.

Selection follows strict rules. If the same prefix is learned from OSPF (default preference 10) and BGP (default preference 170), the OSPF route wins because its preference value is lower (more preferred). If several paths come from the same protocol (for example OSPF), the one with the best metric wins.

```
user@R1> show route

inet.0: 26 destinations, 26 routes (26 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.1.2.0/24        *[Direct/0] 12:44:40
                    > via ge-0/0/0.0
192.168.1.5/32     *[IS-IS/18] 12:41:31, metric 20
                    > to 10.1.2.2 via ge-0/0/0.0
203.0.113.0/24     *[BGP/170] 01:10:15, localpref 100
                    > to 10.1.2.2 via ge-0/0/0.0
```

### 1.2 The forwarding table (FIB)

The **forwarding table** (Forwarding Information Base, FIB) holds only the best prefixes selected by the control plane. It is the table the data plane, the **Packet Forwarding Engine (PFE)**, uses to make high-speed forwarding decisions.

The FIB is streamlined: just enough information to process a packet (destination prefix, next-hop IP, outgoing interface). It doesn't record which protocol originated the route, because forwarding doesn't need that. On hardware-based routers the FIB is programmed directly into ASICs for line-rate performance.

**Juniper vs. Cisco command equivalence**

| Plane | Juniper command | Cisco command |
|---|---|---|
| Control plane (RIB) | `show route` | `show ip route` |
| Data plane (FIB) | `show route forwarding-table` | `show ip cef` |

### 1.3 Refresher: BGP next-hop resolution

A core idea behind MPLS is how BGP resolves next hops. When a router learns a route from an IBGP peer, the **protocol next hop** is usually the loopback address of a distant router, which is not directly connected.

To forward traffic the router performs a **recursive lookup**: it first finds a path to the BGP protocol next hop using another route in its table, almost always an IGP route (OSPF or IS-IS). That IGP route supplies the immediate, physical next hop.

```
[Site A] ---> [R1] --- [R2] --- [R3] --- [R4] --- [R5] ---> [Site B]
                |                                  |
                <---- Service Provider Core ------->
                    (OSPF or IS-IS running inside)
                   (IBGP session between R1 and R5)
```

R1 learns the Site B prefix from its IBGP peer R5. R5's advertisement says "to reach Site B, send packets to me at my loopback". R1 can't send directly to R5's loopback: it looks up R5's loopback, finds the OSPF/IS-IS route to it, and sees that the immediate next hop is R2.

### 1.4 Verifying BGP next hops

`show route detail` shows the two-step process. Note the difference between the **Protocol next hop** (the BGP peer's address) and the physical **Next hop** (the immediate router and interface provided by the IGP).

```
user@R1> show route 203.0.113.0/24 detail

inet.0: 27 destinations, 27 routes (27 active, 0 holddown, 0 hidden)
203.0.113.0/24 (1 entry, 1 announced)
*BGP    Preference: 170/-101
        Next hop type: Indirect
        Protocol next hop: 192.168.1.5          (R5's loopback)
        Source: 192.168.1.8                     (Route Reflector IP)
        ...
        Next hop: 10.1.2.2 via ge-0/0/0.0       (Physical next hop towards R2)
```

Confirm the recursive lookup by checking the route to the protocol next hop itself:

```
user@R1> show route 192.168.1.5

inet.0: 27 destinations, 27 routes (27 active, 0 holddown, 0 hidden)
192.168.1.5/32 (1 entry, 1 announced)
*IS-IS  Preference: 18, metric 40
        > to 10.1.2.2 via ge-0/0/0.0
```

This hop-by-hop resolution, where every router in the core needs the BGP route plus an IGP route to resolve it, is fundamental. MPLS gives you a powerful way to shortcut it.

#### 1.4.1 Route reflector source clarification

In `show route detail` the **Source** field may show the IP address of a route reflector (RR). It identifies the BGP speaker that sent the route to the router you're checking.

It's useful for understanding how the route was advertised, but the RR source IP is **not** used for forwarding. Forwarding depends only on the protocol next hop and the resolved physical next hop.

### 1.5 Key takeaways

> [!IMPORTANT]
> A router keeps two key structures: the **RIB** (control plane, all learned routes) and the **FIB** (data plane, only the best routes, used for forwarding). Forwarding BGP traffic across a core requires a **recursive lookup** in which the BGP next hop is resolved through an IGP.

With these mechanics refreshed, the next module looks at the problems this model created and how MPLS was conceived to solve them.

---

## Module 2: The "Why" of MPLS: Motivations

**Objective:** explain the original performance motivation for MPLS and the modern use cases (traffic engineering, BGP-free cores, VPNs) that make it indispensable in service provider networks today.

### 2.1 Original motivation: the performance problem

Early routers were far less powerful. Layer 2 switching (MAC addresses) could be done in hardware, but Layer 3 routing (IP addresses) was a slow, software-based process: every packet needing a route lookup was "punted" to the router's general-purpose CPU.

As routing tables grew, even to a few thousand prefixes (tiny by today's standards), CPU-based lookup became a major bottleneck. The search algorithms were inefficient and got slower as tables grew. The challenge: speed up forwarding without a full IP lookup at every hop.

```
[Packet In] ---> (Router CPU) --slow--> [Packet Out]
                     |
               [IPv4 Routing Table]
               [  7,500 prefixes  ]
               [ This takes a while... ]
```

### 2.2 The proposed solution: labels

Engineers noticed that even with thousands of prefixes, traffic leaving a router was usually headed to only a few possible next hops. It was wasteful for every router on the path to repeat the same slow IP lookup only to reach one of a handful of outcomes.

The idea: do the complex IP lookup **once, at the network edge**. The ingress router attaches a simple numeric **label** to the packet. Every router after it in the core (transit routers) ignores the IP header and forwards based only on the label. This pre-signalled, end-to-end path for labelled packets is a **Label-Switched Path (LSP)**, effectively a tunnel across the network.

That is the core of **Multiprotocol Label Switching (MPLS)**. Packets are switched on labels, and because core routers never inspect the inner packet, the payload can be any protocol (IPv4, IPv6, Ethernet): hence "multiprotocol".

#### 2.2.1 Historical context: replacing ATM and Frame Relay

MPLS was developed in the late 1990s to address the performance limits of pure IP routing and to unify legacy technologies such as **ATM (Asynchronous Transfer Mode)** and **Frame Relay**. These gave reliable, circuit-like forwarding using virtual circuits with fixed-size cells (ATM) or variable-length frames (Frame Relay).

MPLS kept many of their advantages (fast forwarding on labels instead of destination lookups) while letting providers move to an IP-based core without losing the traffic engineering and VPN capabilities that ATM and Frame Relay offered.

#### 2.2.2 Label operations: push, swap, pop

| Operation | What it does | Where it usually happens |
|---|---|---|
| **Push** | Adds a new label to the packet | Ingress PE, to start the LSP |
| **Swap** | Replaces the incoming label with a different outgoing label | Transit P routers in the core (the most common operation) |
| **Pop** | Removes the label so normal IP forwarding can continue | Egress PE, or the penultimate hop (PHP) |

These operations allow efficient forwarding without inspecting the IP header at every hop.

#### 2.2.3 Label distribution protocols: LDP and RSVP-TE

Routers use label distribution protocols to advertise and manage the label bindings that build LSPs:

- **LDP (Label Distribution Protocol):** distributes labels automatically based on the existing IGP routing table, so LSPs follow the normal IGP best path (best-effort LSPs).
- **RSVP-TE (Resource Reservation Protocol, Traffic Engineering):** supports explicit path setup and reservation of resources (bandwidth, constraints), enabling MPLS traffic engineering.

Both are covered in detail in later parts on LSP signalling and traffic engineering.

### 2.3 Modern motivation 1: the BGP-free core

Ironically, while MPLS was being developed, ASICs and lookup algorithms improved so quickly that the original performance problem disappeared: routers could do IP lookups at line rate.

MPLS found new uses. One of the most popular is the **BGP-free core**. In a traditional design every core router runs BGP to resolve next hops, which needs expensive routers with lots of memory and CPU everywhere.

With MPLS, only the edge routers (**Provider Edge, PE**) run BGP. The core routers (**Provider, P**) only switch labels. They never look at customer IP packets, so they don't need the BGP table. Providers can build a fast core from cheaper P routers optimised for throughput and put the routing intelligence and complex features on more powerful PEs at the edge.

```
(Customer 1)                                        (Customer 1)
 [Site A]                                            [Site B]
     |                                                   |
+----|---------------------------------------------------|----+
|    |               Service Provider Core               |    |
|  +----+      +----+      +----+      +----+      +----+     |
|  | R1 |------| R2 |------| R3 |------| R4 |------| R5 |     |
|  |(PE)|      | (P)|      | (P)|      | (P)|      |(PE)|     |
|  +----+      +----+      +----+      +----+      +----+     |
|    |   <-- P routers talk only IGP + MPLS (small RIB) -->    |
|    |       (high speed, lower cost hardware)           |    |
+----|---------------------------------------------------|----+
     |                                                   |
 [Site A]                                            [Site B]
(Customer 2)                                        (Customer 2)
```

### 2.4 Modern motivation 2: traffic engineering

In a pure IP network traffic follows the best IGP path. It's all-or-nothing: you can't easily send some traffic one way and other traffic another way without complex policy-based routing.

**MPLS traffic engineering (MPLS-TE)** fixes this. You can build explicit LSPs that don't follow the IGP shortest path, including several LSPs between the same two endpoints, each over a different physical route.

For example: a low-latency LSP over the shortest path for high-priority voice and video, and a longer, less congested or cheaper path for best-effort traffic. LSPs can carry constraints such as required bandwidth or links to avoid.

```
   Priority traffic LSP:    R1 -> R2 -> R5

        [R1]-------[R2]-------[R5]
          |       /    \       |
          '---[R3]------[R4]---'

   Best-effort LSP:         R1 -> R3 -> R4 -> R5
```

### 2.5 Other modern use cases

Because MPLS tunnels traffic transparently, it enables many other services:

- **VPN services:** MPLS is the foundation for Layer 3 VPNs (L3VPN) and Layer 2 VPNs (VPLS, pseudowires). A provider can carry thousands of customers over a shared core with each customer fully isolated, even with overlapping IP addresses.
- **IPv6 adoption:** an ISP can offer IPv6 without running IPv6 on every core router. Edge routers carry IPv6 packets inside an MPLS LSP across an IPv4-only core.
- **Multicast traffic:** MPLS can build **point-to-multipoint (P2MP) LSPs** that replicate traffic from one source to many destinations. Ideal for IPTV, and removes the need for protocols like PIM in the core.

### 2.6 Terminology: MPLS vs. MPLS VPN

"MPLS" has two meanings, which causes confusion:

| Meaning | Refers to |
|---|---|
| **The technology** | The transport mechanism: labels, LSPs and the protocols that signal them (LDP, RSVP). This is the focus of these notes. |
| **The service** | Especially in enterprises, "an MPLS circuit" often means an MPLS Layer 3 VPN service bought from a provider. |

Mixing the two is a **fallacy of equivocation**. A debate like "MPLS vs. SD-WAN" gets confused when one person means the transport and the other means the VPN service. Be precise: say **"MPLS VPN"** for the service and **"MPLS"** for the transport.

### 2.7 Key takeaways

> [!IMPORTANT]
> MPLS was conceived to solve a hardware performance problem that no longer exists, yet it has become a cornerstone of modern networks. It enables scalable designs like the **BGP-free core**, **traffic engineering**, and services such as **VPNs** and **IPv6 tunnelling**.

---

## Module 3: Next Steps in Your MPLS Journey

With the fundamentals in place, continue with:

- **MPLS LDP and RSVP-TE configuration:** building label-switched paths and understanding signalling details.
- **MPLS Layer 3 VPNs (RFC 4364):** using MPLS to deliver customer VPN services across the provider core.
- **MPLS Layer 2 VPNs (VPLS, pseudowires):** Ethernet-like services across an MPLS core.

These build on the MPLS foundation and prepare you for the more advanced topics in the JNCIS-SP and JNCIP-SP tracks.

---

## Module 4: Glossary of Key Terms

| Term | Meaning |
|---|---|
| **MPLS** (Multiprotocol Label Switching) | Forwarding technology that moves data between nodes using short path labels instead of long network addresses, avoiding complex routing-table lookups. |
| **LSP** (Label-Switched Path) | A pre-determined path, or tunnel, through an MPLS network that packets with a given label follow. |
| **ASIC** (Application-Specific Integrated Circuit) | Specialised chip in the router's PFE that performs high-speed forwarding lookups in hardware. |
| **DFZ** (Default-Free Zone) | The set of internet routers that carry the full global BGP table and need no default route. |
| **PDU** (Protocol Data Unit) | Generic term for one unit of information exchanged between peers (for example an Ethernet frame or an IP packet). |
| **SD-WAN** (Software-Defined WAN) | WAN approach using centralised control to steer traffic across the WAN based on application needs, network conditions and security policy. |
| **ATM** (Asynchronous Transfer Mode) | Legacy switching technology using small fixed-size cells. One of the technologies MPLS was designed to integrate with and eventually replace. |
| **P2MP LSP** (Point-to-Multipoint LSP) | An LSP that replicates packets from one ingress router to several egress routers, delivering multicast-like traffic without multicast routing protocols in the core. |

---

> 🧠 **Test yourself:** [Recall guide for this topic](../recall/R01-mpls-fundamentals-recall.md)

[Index](../README.md) · [Next: The Mechanics of MPLS](02-mpls-mechanics.md) →
