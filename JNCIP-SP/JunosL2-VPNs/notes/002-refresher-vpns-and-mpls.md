# Refresher: VPNs and MPLS

**VPN types, PE/P/CE roles, the MPLS data plane, the two-label stack, and L3VPN vs. L2VPN**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · JNCIP-SP · Junos Layer 2 VPNs · Part 002

← [Previous: Introduction to Junos Layer 2 VPNs](001-introduction-to-junos-l2vpns.md) · [Index](../README.md) · [Next: The Different Flavors of Layer 2 VPN](003-flavors-of-layer-2-vpn.md) →

**Contents:** [1 VPNs](#module-1-vpn-technology-overview) · [2 MPLS Refresher](#module-2-mpls-fundamentals-refresher) · [3 Data Plane](#module-3-the-mpls-data-plane-in-action) · [4 Two-Label Stack](#module-4-introduction-to-mpls-vpns-and-the-two-label-stack) · [5 L3VPN vs. L2VPN](#module-5-comparing-mpls-layer-3-and-layer-2-vpns) · [6 Practice](#module-6-exam-practice-questions) · [7 Glossary](#module-7-glossary) · [8 Dynamic Recall](#module-8-dynamic-recall)

---

## Module 1: VPN Technology Overview

**Objectives:**
- Define a Virtual Private Network (VPN) and its core purpose.
- Differentiate between the three primary VPN technologies: IPsec, SD-WAN and MPLS.

### 1.1 What is a VPN?

A VPN, or Virtual Private Network, is any technology that connects private networks together across a separate, often public, third-party network. The goal is to make all connected sites behave as one large, private network, allowing devices to communicate using their internal IP addresses as if they were in the same building. The network in the middle is completely transparent to the end-user devices.

```
                        +-----------------------------+
   [Corporate HQ] ------|                             |------ [Data Center]
                        |   Someone else's network    |
   [Branch Office] -----|  (Internet or provider core)|------ [Home Office]
                        |                             |
                        +-----------------------------+

   All sites behave as one private network and talk using their internal
   IP addresses. The network in the middle is invisible to them.
```

### 1.2 Comparing VPN solutions

In today's networks you'll mainly encounter three types of VPN solution, each with distinct advantages and use cases.

| VPN type | What it is | Strengths | Trade-offs |
|---|---|---|---|
| **1. IPsec VPNs** | Short for IP Security. Creates secure, encrypted and authenticated tunnels between sites. | The standard choice for connecting sites over the public Internet. | Can require powerful hardware for encryption, and can carry administrative overhead in large deployments. |
| **2. SD-WAN (Software-Defined WAN)** | A centralized controller automates the creation and management of VPN tunnels (often using IPsec). | Simplifies large-scale deployments and adds intelligent path selection. | Some customers prefer not to manage their own SD-WAN solution. |
| **3. MPLS VPNs** | A service provider-managed solution: the customer offloads all VPN complexity to the provider, which uses its MPLS core to build private VPNs for its customers. | Powerful, scalable, managed by the provider. | Not a solution for building your own tunnels over the public Internet. |

This course focuses exclusively on MPLS VPNs.

**Knowledge check.** A company needs to securely connect its branch offices over the public Internet and wants to manage the solution itself. Which VPN technology is the most appropriate choice?

- A) MPLS VPN
- B) IPsec VPN
- C) A direct fiber connection

<details><summary>Answer</summary>

**B.** IPsec is the standard for creating secure tunnels over the public Internet, a scenario where MPLS VPNs (a private service) are not applicable.

</details>

> [!IMPORTANT]
> **Key takeaways.** VPNs are essential for connecting geographically separate sites into a single private network. While IPsec and SD-WAN are popular solutions, many customers prefer to outsource this complexity to a service provider using an MPLS VPN. To understand how MPLS VPNs work, we first need to refresh the fundamentals of MPLS itself.

---

## Module 2: MPLS Fundamentals Refresher

**Objectives:**
- Define Multiprotocol Label Switching (MPLS) and Label Switched Path (LSP).
- Explain the roles of PE, P and CE routers in a service provider network.
- Understand the function of transport labels and the concept of a BGP-free core.

### 2.1 Core MPLS concepts

MPLS (Multiprotocol Label Switching) is a forwarding technique where routers make decisions based on a short, fixed-length **label** attached to a packet, rather than performing a longest-match lookup on the destination IP address. Traffic crosses an MPLS network through a tunnel called a **Label Switched Path (LSP)**.

A label is simply a number (20 bits, so 0 to 1,048,575) that acts as an instruction. When a router receives a packet with a specific label, it knows exactly what to do with it, for example: "swap this label for a new one and send the packet out of interface ge-0/0/1."

> [!NOTE]
> For the label header in detail (label, TC, S bit and TTL) and the reserved labels 0 to 15, see [MPLS mechanics](../../../Junos-MPLS-Fundamentals/notes/02-mpls-mechanics.md).

### 2.2 Service provider terminology: PE, P and CE

In an MPLS network, routers have specific roles:

```
  Customer                         Provider network                        Customer
 +--------+     +--------+     +-----+     +-----+     +-----+     +--------+     +--------+
 |  CE-1  |-----|  PE-1  |-----| P-1 |-----| P-2 |-----| P-3 |-----|  PE-2  |-----|  CE-2  |
 +--------+     +--------+     +-----+     +-----+     +-----+     +--------+     +--------+
  Customer       Provider      Provider    Provider    Provider     Provider       Customer
  Edge           Edge          (core)      (core)      (core)       Edge           Edge
                 |<--- LSP starts                        LSP ends --->|
                 |<-------------- BGP between PEs only -------------->|
```

| Role | Description |
|---|---|
| **CE (Customer Edge)** | A device at the customer's site (for example a router or firewall) that connects to the service provider. |
| **PE (Provider Edge)** | A router at the edge of the provider's network. It connects to customers, speaks BGP with other PEs, and acts as the ingress (start) and egress (end) point of LSPs. |
| **P (Provider)** | A core/transit router within the provider's network. Its only job is to forward packets based on their MPLS label, without ever looking at the customer's payload. This allows a **"BGP-free core"**: P routers don't need to run BGP or hold massive Internet routing tables, reducing state and complexity. |

**Knowledge check.** Match the router type to its primary function in an MPLS network:

| Router | Function |
|---|---|
| 1. PE router | A) Forwards packets based only on the MPLS label. |
| 2. P router | B) Connects to the service provider network. |
| 3. CE router | C) Starts and ends LSPs and connects to customers. |

<details><summary>Answer</summary>

1-C, 2-A, 3-B

</details>

> [!IMPORTANT]
> **Key takeaways.** MPLS uses labels to forward traffic through LSPs. The network is made of intelligent PE routers at the edge and simple, fast P routers in the core. This design allows a scalable "BGP-free core." Now that we understand the roles, let's see how a packet actually moves through this network.

---

## Module 3: The MPLS Data Plane in Action

**Objectives:**
- Explain the three fundamental MPLS label operations: push, swap and pop.
- Follow a packet's journey across a Label Switched Path (LSP).
- Define Penultimate Hop Popping (PHP).

### 3.1 A packet's journey

Let's trace a packet from a customer site, across the service provider core, to another customer site. This process involves three key actions on the MPLS label.

```
 [Peer A] --IP--> [PE-1] --L 123456--> [P-1] --L 234567--> [P-2] --IP--> [PE-2] --IP--> [Peer B]
                   PUSH                 SWAP                 POP
                (ingress LER)          (transit)      (penultimate hop)   (egress LER)

 PE-2 advertised label 3 (implicit null) for itself, which tells P-2 to pop.
```

1. **PUSH:** an IP packet arrives at the ingress router, PE-1. PE-1 determines the packet needs to go to PE-2 through an LSP. It **pushes** the first MPLS label (for example `123456`) onto the packet and forwards it to the first P router, P-1.
2. **SWAP:** P-1 receives the packet. It looks only at the label `123456`. Its table (`mpls.0` in Junos) says this label means "swap the label to `234567` and forward to P-2". It performs the swap and sends the packet on. This swap happens at every P router in the path, up to the penultimate hop.
3. **POP:** the packet arrives at P-2, the last P router before the destination. This router is the **penultimate hop**. It knows PE-2 doesn't need the transport label, so it simply **pops** (removes) the label and forwards the original IP packet to PE-2. This is called **Penultimate Hop Popping (PHP)**, and it saves the egress PE from having to do an extra MPLS lookup before its IP lookup.

> [!NOTE]
> P-2 doesn't decide on its own to pop. The egress router (PE-2) signals it by advertising **label 3 (implicit null)** for its own address, through LDP or RSVP. PHP is the Junos default for both. You can turn it off with `explicit-null` (label 0 is then sent to the egress for IPv4). See [MPLS mechanics](../../../Junos-MPLS-Fundamentals/notes/02-mpls-mechanics.md) for the details.

> [!IMPORTANT]
> **Key takeaways.** The MPLS data plane is a simple yet powerful process: push a label at the start, swap it at each hop in the core, and pop it at the end (normally at the penultimate hop). This lets P routers forward traffic at high speed without ever inspecting the IP header. This transport mechanism is so generic that it can carry anything, which leads us to MPLS VPNs.

---

## Module 4: Introduction to MPLS VPNs and the Two-Label Stack

**Objectives:**
- Explain how LSPs can tunnel any type of traffic, not just IPv4.
- Define the function of a VPN (or service) label.
- Describe the structure of the two-label stack used in MPLS VPNs.

### 4.1 LSPs: the universal tunnel

Because P routers only look at the outermost label, the payload inside the LSP is completely invisible to them. This means an LSP can carry any kind of traffic, including IPv6, multicast, or even raw Ethernet frames. This flexibility is what makes MPLS VPNs possible.

However, this raises a question: if a PE router is connected to hundreds of different customers, how does it know which customer's private network the traffic belongs to when it arrives at the other end of the LSP? The answer is a second label.

### 4.2 The two-label stack

MPLS VPNs use a stack of two labels, both to transport the packet across the core and to identify which VPN it belongs to.

```
      Layer 3 VPN packet                         Layer 2 VPN packet
 +---------------------------------+      +---------------------------------+
 | Service provider L2 header      |      | Service provider L2 header      |
 +---------------------------------+      +---------------------------------+
 | Outer transport label           |      | Outer transport label           |  <-- Used by P routers.
 +---------------------------------+      +---------------------------------+      Changes hop by hop.
 | Inner VPN label (bottom, S=1)   |      | Inner VPN label (bottom, S=1)   |  <-- Used by egress PE.
 +---------------------------------+      +---------------------------------+      Identifies the customer
 | Customer IP header              |      | (Optional control word)         |      VRF / pseudowire.
 +---------------------------------+      +---------------------------------+
 | Customer TCP header             |      | Customer Ethernet frame         |
 +---------------------------------+      | (MAC header, VLAN tag, payload) |
 | Customer payload                |      +---------------------------------+
 +---------------------------------+
```

- **Outer transport label:** the label we've already discussed. The P routers use it to get the packet from the ingress PE to the egress PE. It is swapped at every hop (and normally popped at the penultimate hop).
- **Inner VPN label (or service label):** added by the ingress PE, underneath the transport label. It stays on the packet for the entire journey across the core. When the packet arrives at the egress PE, this label tells the PE which specific customer VPN (in an L3VPN, which routing table or VRF) the packet belongs to.

> [!NOTE]
> Two refinements to keep in mind for the rest of this course:
> - Because of PHP, the egress PE usually receives the packet with **only the inner VPN label** left on it. That is exactly what the story in [Module 8](#module-8-dynamic-recall) is about.
> - In a **Layer 2 VPN** there is no VRF. The inner label identifies a **pseudowire**, and so the local attachment circuit (the CE-facing interface) the frame must be sent out of. The egress PE forwards the frame without any IP lookup. In a Junos L3VPN the VPN label, by default, also points straight at the outgoing CE next hop (per-next-hop label allocation), unless you configure `vrf-table-label`, which makes the egress PE do an IP lookup in the VRF.

> [!IMPORTANT]
> **Key takeaways.** MPLS VPNs rely on a two-label stack. The outer label provides transport, while the inner label provides service identification. This simple mechanism lets service providers host thousands of logically separate customer networks on a single, shared infrastructure. Now let's see how this model is applied to deliver Layer 3 versus Layer 2 services.

---

## Module 5: Comparing MPLS Layer 3 and Layer 2 VPNs

**Objectives:**
- Differentiate between the service models of Layer 3 and Layer 2 VPNs.
- Understand the key characteristics of an L3VPN, including the VRF.
- Understand the key characteristics of an L2VPN (virtual wire or virtual switch).

### 5.1 Layer 3 VPNs (L3VPN)

In a Layer 3 VPN, the service provider **participates in the customer's Layer 3 routing**. The PE router learns the customer's IP prefixes (for example through BGP, OSPF or static routes) and the CE router sees the PE as its next-hop router.

To keep customers separate, each customer's circuit is placed into a unique routing instance called a **VRF (Virtual Routing and Forwarding)** table. A VRF is like a virtual router inside the PE, with its own separate routing table. This is how two different customers can use exactly the same private address space (for example 192.168.1.0/24) without any conflict.

### 5.2 Layer 2 VPNs (L2VPN)

In a Layer 2 VPN, the service provider **does not participate in customer routing**. Instead, the provider's network acts as a giant, virtual Ethernet cable or switch, extending the customer's Layer 2 domain between sites. The customer's routers see each other as if they were directly connected.

```
 Virtual wire (point-to-point)

   [Site A] ======== SP core acts as a virtual wire ======== [Site B]


 Virtual switch (point-to-multipoint)

   [Site A] ---+                                +--- [Site B]
               |     +--------------------+     |
               +-----|  SP core acts as a |-----+
               |     |   virtual switch   |     |
   [Site C] ---+     |  (learns MACs)     |     +--- [Site D]
                     +--------------------+
```

There are two main models:

| Model | Connects | Behaviour | Technologies |
|---|---|---|---|
| **Virtual wire (point-to-point)** | Exactly two customer sites | Whatever enters one end comes out of the other; no MAC learning | BGP L2VPN, LDP L2Circuits |
| **Virtual switch (point-to-multipoint)** | Multiple customer sites in a single broadcast domain | Learns MAC addresses like a real switch | VPLS, EVPN |

> [!IMPORTANT]
> **Key takeaways.** The fundamental difference is the layer at which the provider interacts with the customer. L3VPNs are a routed service (PEs are a Layer 3 hop), while L2VPNs are a switched service (PEs are transparent at Layer 3). The rest of this course dives into the technologies used to build these Layer 2 VPN services.

---

## Module 6: Exam Practice Questions

**Question 1.** In an MPLS VPN, what is the primary function of the inner label?

- A) To forward the packet from one P router to the next.
- B) To identify which customer VRF the packet belongs to at the egress PE.
- C) To provide encryption for the customer's payload.
- D) To ensure Penultimate Hop Popping occurs.

<details><summary>Answer</summary>

**B.** The inner label is the VPN (service) label. Its purpose is to tell the egress PE which customer VPN the packet belongs to: in an L3VPN, which VRF (or, with Junos default per-next-hop labels, directly which CE next hop); in an L2VPN, which pseudowire and attachment circuit.

</details>

**Question 2.** Which of the following is a key characteristic of a Layer 3 VPN that distinguishes it from a Layer 2 VPN?

- A) It uses a two-label stack.
- B) The service provider's PE routers participate in the customer's routing.
- C) It can connect more than two sites.
- D) It uses an MPLS transport core.

<details><summary>Answer</summary>

**B.** In an L3VPN, the PE-CE relationship is a Layer 3 routing adjacency. In an L2VPN, the provider network is transparent at Layer 3. (Both use a two-label stack and an MPLS core, and VPLS/EVPN L2VPNs connect more than two sites too.)

</details>

---

## Module 7: Glossary

| Term | Definition |
|---|---|
| **VPN (Virtual Private Network)** | A technology that connects private networks across a separate third-party network, making them function as a single network. |
| **MPLS (Multiprotocol Label Switching)** | A forwarding method that uses labels instead of IP address lookups to forward traffic. |
| **LSP (Label Switched Path)** | A unidirectional tunnel or path through an MPLS network. |
| **PE router (Provider Edge)** | A router at the edge of a service provider network that connects to customers and manages VPN services. |
| **P router (Provider)** | A core router in a service provider network that performs high-speed label swapping. |
| **CE router (Customer Edge)** | A device at the customer premises that connects to the PE router. |
| **Transport label** | The outer label in an MPLS stack, used to forward a packet from the ingress PE to the egress PE. |
| **VPN label (service label)** | The inner label in an MPLS stack, used by the egress PE to identify the specific customer VPN (VRF or pseudowire). |
| **VRF (Virtual Routing and Forwarding)** | A virtual router instance within a PE that holds a unique, separate routing table for a specific L3VPN customer. |
| **PHP (Penultimate Hop Popping)** | An optimization where the second-to-last router in an LSP removes the transport label before forwarding the packet to the final router. Triggered by the egress advertising label 3 (implicit null). |

---

## Module 8: Dynamic Recall

### Part 1: Recall

Test your knowledge before opening each answer.

**What are the three primary types of VPNs commonly encountered today?**

<details><summary>Answer</summary>

**IPsec VPNs** (for security over the public Internet), **SD-WAN** (for automated management) and **MPLS VPNs** (as a managed service provider offering).

</details>

**What is the fundamental difference between a P router and a PE router?**

<details><summary>Answer</summary>

A **P (Provider) router** is a core transit device that only forwards traffic based on MPLS labels, enabling a "BGP-free core". A **PE (Provider Edge) router** is an intelligent edge device that connects to customers, manages VPNs, and performs label push/pop operations.

</details>

**What are the three main MPLS label operations in the data plane?**

<details><summary>Answer</summary>

**Push:** the ingress PE adds the first label. **Swap:** P routers change the label at each hop. **Pop:** the penultimate hop router removes the label before sending to the egress PE (this is PHP).

</details>

**What are the distinct roles of the two labels in an MPLS VPN label stack?**

<details><summary>Answer</summary>

The **outer (transport) label** is used by P routers to forward the packet across the core to the correct egress PE. The **inner (VPN/service) label** is used by the egress PE to identify which specific customer VPN (VRF, or pseudowire in an L2VPN) the packet belongs to.

</details>

**What is the core difference between an MPLS Layer 3 VPN and a Layer 2 VPN?**

<details><summary>Answer</summary>

In a **Layer 3 VPN**, the provider participates in customer routing using VRFs. In a **Layer 2 VPN**, the provider is transparent at Layer 3 and acts as a virtual wire or switch to extend the customer's L2 domain.

</details>

### Part 2: Fusion

> [!TIP]
> Naz stared at the packet capture, completely baffled. "The packet is arriving at the destination PE router, but it's getting dropped. I can see it has a label, but it's not being forwarded to the customer. Why?"
>
> His senior engineer, Maria, glanced over. "Which label are you looking at?" she asked. Naz pointed. "This one, the transport label. It got the packet all the way across the core."
>
> "Ah," Maria said with a smile. "You're forgetting the P router's gift. The **penultimate hop** popped that transport label before it even got to the PE. What you're seeing is the **inner VPN label**. The PE router has no idea what to do with that label unless it's configured for that specific customer's **VRF**. The P routers did their job perfectly: they're the postal service, just moving mail. But the PE is the mailroom clerk who needs to know which person's mailbox (which VRF) the letter goes into. No VRF for that VPN label means the letter goes in the bin."
>
> It clicked for Naz. The P routers were blissfully ignorant of the payload, just doing their label-swapping job. The PE needed two pieces of information: the transport path (which the LSP provides) and the customer context (which the VPN label provides). Without both, the service fails.

### Part 3: Chunk and collapse

| Chunk | One-sentence summary | Tags |
|---|---|---|
| **VPNs** | VPNs create a single, private network for a customer across a separate, third-party network. | #VirtualPrivate #IPsec #SD-WAN #MPLS |
| **MPLS roles** | PEs are the smart edge, Ps are the simple core, and CEs are the customer's gear. | #EdgeVsCore #BGP-FreeCore #PE-P-CE #ProviderVsCustomer |
| **The two-label stack** | The outer label is for transport across the core, while the inner label identifies the specific VPN service at the destination. | #Transport+Service #OuterVsInner #MailSorting #LabelStack |
| **L3VPN vs. L2VPN** | L3VPNs are a routed service where the provider manages customer routes in a VRF; L2VPNs are a switched service that extends the customer's LAN. | #RoutedVsSwitched #VirtualRouter #VirtualWire #LayerOfInteraction |

---

← [Previous: Introduction to Junos Layer 2 VPNs](001-introduction-to-junos-l2vpns.md) · [Index](../README.md) · [Next: The Different Flavors of Layer 2 VPN](003-flavors-of-layer-2-vpn.md) →
