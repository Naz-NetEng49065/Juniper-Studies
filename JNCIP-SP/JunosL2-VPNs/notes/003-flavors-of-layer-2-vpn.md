# The Different Flavors of Layer 2 VPN

**Virtual wires vs. virtual switches: pseudowires, their signaling options, VPLS and EVPN**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · JNCIP-SP · Junos Layer 2 VPNs · Part 003

← [Previous: Refresher: VPNs and MPLS](002-refresher-vpns-and-mpls.md) · [Index](../README.md) · [Next: L2VPN: BGP-Signaled Pseudowires](004-bgp-signaled-l2vpn.md) →

**Contents:** [1 Two Models](#module-1-the-two-models-of-layer-2-vpns) · [2 Pseudowires](#module-2-pseudowires-the-virtual-wire) · [3 Signaling](#module-3-signaling-the-pseudowire) · [4 VPLS](#module-4-vpls-the-virtual-switch) · [5 EVPN](#module-5-evpn-the-modern-virtual-switch) · [6 Practice](#module-6-exam-practice-questions) · [7 Glossary](#module-7-glossary) · [8 Dynamic Recall](#module-8-dynamic-recall)

---

## Module 1: The Two Models of Layer 2 VPNs

**Objectives:**
- Differentiate between the "virtual wire" and "virtual switch" service models.
- Identify the core technologies associated with each model.

### 1.1 A tale of two services

Layer 3 VPNs offer a single, unified service: a distributed virtual router. Layer 2 VPNs are split into two fundamentally different service models. Which one you use depends on whether the customer needs to connect just two sites or multiple sites.

#### Model 1: The virtual wire

```
  +--------+                                           +--------+
  | Site A |===========( Virtual Wire )================| Site B |
  +--------+     one logical "cable" through the SP    +--------+
```

A **point-to-point** service that acts like a long Ethernet cable connecting exactly two customer sites. The service provider is completely transparent, and no MAC address learning is needed. This is built with **pseudowires**.

#### Model 2: The virtual switch

```
  +--------+                             +--------+
  | Site A |---+                     +---| Site B |
  +--------+   |   +-------------+   |   +--------+
               +---|   Virtual   |---+
               +---|   Switch    |---+
  +--------+   |   | (SP network)|   |   +--------+
  | Site C |---+   +-------------+   +---| Site D |
  +--------+                             +--------+
        All sites share one broadcast domain
```

A **point-to-multipoint** service that acts like a distributed Ethernet switch, connecting two or more sites into the same broadcast domain. The provider's network learns MAC addresses to forward traffic correctly. This is built with **VPLS** or **EVPN**.

### 1.2 Key takeaways

> [!IMPORTANT]
> Layer 2 VPNs are not a single technology but a category of services split into two models: point-to-point **virtual wires** (pseudowires) and point-to-multipoint **virtual switches** (VPLS/EVPN). Understanding this distinction is the first step to mastering L2VPNs. Next: a deep dive into the virtual wire model.

---

## Module 2: Pseudowires (The Virtual Wire)

**Objectives:**
- Define a pseudowire and its alternative name, Virtual Private Wire Service (VPWS).
- Explain the concept of an attachment circuit.
- Describe how VLAN tags can multiplex multiple pseudowires on a single physical link.
- List common use cases for pseudowires.

### 2.1 Pseudowire fundamentals

A **pseudowire** creates a logical, virtual wire across a service provider's network, connecting two customer sites. From the customer's point of view, their two CE devices are joined by a long Ethernet cable. The provider network is entirely transparent: if you ran LLDP between the two CEs, they would discover each other directly.

Because it emulates a physical wire, this service is also called **Virtual Private Wire Service (VPWS)**.

### 2.2 Attachment circuits and VLANs

The logical connection from a customer's CE to the provider's PE is called an **attachment circuit (AC)**. Think of it as a *logical* connection: one physical interface on a PE can host many attachment circuits, usually told apart by VLAN tags.

This gives two deployment models:

| Model | How it works |
|---|---|
| **Single logical unit** | The whole physical interface is dedicated to one pseudowire. Any VLAN tags on incoming customer frames are treated as payload and carried transparently across the wire. |
| **Multiple logical units** | The physical interface acts as a trunk. Each incoming VLAN tag identifies a different customer or remote site, and the PE maps the frame to the matching pseudowire. A single hub site can have its own virtual wire to many spoke sites. |

```
                                +--- PW to Site 2 (VLAN 200) --- [PE-2] --- [Site 2]
                                |
  [Hub Site 1] --- [PE-1] ======+   (one physical trunk, one AC per VLAN)
                                |
                                +--- PW to Site 3 (VLAN 300) --- [PE-3] --- [Site 3]
```

> [!NOTE]
> In Junos terms, the single-unit model is typically `encapsulation ethernet-ccc` on the physical interface with `unit 0 family ccc`. The multi-unit model uses `vlan-tagging` (or `flexible-vlan-tagging`) with per-unit `vlan-ccc` / `extended-vlan-ccc` encapsulation, or `flexible-ethernet-services` when you mix CCC and non-CCC units on one port. The configuration notes cover this in detail.

### 2.3 Use cases for pseudowires

- **Wholesale services:** a wholesale provider uses a pseudowire to carry a customer's traffic transparently to the PE of the ISP they chose. The customer appears directly connected to their ISP.
- **Site-to-site connectivity:** a simple, cost-effective way to join two customer offices or sites as if they were on the same LAN segment.
- **Data Center Interconnect (DCI):** connecting two data centers (or even two racks in the same data center) at Layer 2 over a Layer 3 fabric.

### 2.4 Key takeaways

> [!IMPORTANT]
> Pseudowires provide a transparent, point-to-point virtual wire. They don't learn MAC addresses; they simply forward every frame from one attachment circuit to the other. But how do the PEs at each end find each other and set up the tunnel? That's the job of a signaling protocol.

---

## Module 3: Signaling the Pseudowire

**Objectives:**
- List the five methods for signaling a pseudowire in Junos.
- Compare the two most common methods: BGP L2VPN and LDP L2Circuit.
- Clarify the often-confused terms "L2VPN", "L2Circuit", "Kompella" and "Martini".

### 3.1 Five ways to build a wire

The customer always gets the same result (a virtual wire), but a provider can use five different control-plane protocols to set up a pseudowire. The choice often comes down to history or operational preference.

| Junos name | Signaling protocol | Key feature | Common nickname | Primary use case |
|---|---|---|---|---|
| **L2VPN** | BGP | Autodiscovery of PEs | Kompella | Large-scale deployments with many sites |
| **L2Circuit** | LDP | Simple, manual configuration | Martini | Simple point-to-point links where simplicity is key |
| **FEC 129** | BGP + LDP | BGP for autodiscovery, LDP for signaling | - | VPLS deployments (less common for pseudowires) |
| **Circuit Cross-Connect (CCC)** | RSVP | Uses a dedicated RSVP LSP | - | Legacy, or hardware with limited label stack depth |
| **EVPN-VPWS** | BGP (EVPN) | Modern, flexible signaling | - | Greenfield deployments needing advanced features |

> [!NOTE]
> CCC fits hardware with limited label stack depth because it needs no inner VPN label: the circuit is mapped straight onto its own RSVP LSP (one per direction), so the frame carries just the transport label. The price is one LSP per circuit.

### 3.2 Clarifying the terminology

The names cause a lot of confusion, because the most common terms each have two meanings.

| Term | Generic meaning | Junos-specific meaning |
|---|---|---|
| **L2VPN** | "layer 2 VPN" (lowercase *l*): the whole category of services that extend Layer 2, including all five methods above | Only pseudowires signaled with **BGP** (Kompella) |
| **L2Circuit** | "layer 2 circuit": often used in the industry as another name for any pseudowire | Only pseudowires signaled with **LDP** (Martini) |

### 3.3 Key takeaways

> [!IMPORTANT]
> There are several ways to signal a pseudowire, each with different operational trade-offs. BGP-signaled **L2VPNs** (Kompella) offer scalable autodiscovery; LDP-signaled **L2Circuits** (Martini) offer simplicity. Knowing the exact Junos terminology avoids a lot of confusion. Next: the virtual switch model.

---

## Module 4: VPLS (The Virtual Switch)

**Objectives:**
- Define Virtual Private LAN Service (VPLS) and its role as a virtual switch.
- Explain VPLS's reliance on data-plane MAC learning (flood-and-learn).
- Identify the key disadvantages of VPLS that led to EVPN.

### 4.1 VPLS: the virtual LAN

If a pseudowire is a virtual wire, **VPLS (Virtual Private LAN Service)** is a virtual LAN. It creates a point-to-multipoint service where the provider network acts as one distributed virtual switch for the customer. All customer sites attached to the VPLS instance talk as if they were plugged into the same physical Ethernet switch.

Under the hood, a VPLS is typically a full mesh of pseudowires between all participating PE routers.

### 4.2 The drawbacks of VPLS

For years VPLS was the standard for multipoint L2 services, but it has significant disadvantages that its successor, EVPN, solves:

- **Data-plane MAC learning:** VPLS learns MAC addresses by inspecting the source of incoming data frames, just like a traditional switch. This flood-and-learn behavior is inefficient and slow. If a MAC moves or an interface goes down, the rest of the network must wait for the old entry to time out, causing a period of packet loss.
- **Active-standby multihoming:** if a customer site connects to two PEs for redundancy, one of the links must be in a blocking state (via Spanning Tree or manual configuration) to prevent loops. The customer can't use all of their bandwidth.
- **Inefficient gateway redundancy:** VPLS relies on VRRP for gateway redundancy, so only one PE holds the active default gateway at a time. Traffic from a site may have to cross the whole SP network to reach the active gateway, even when a closer standby gateway exists.

> [!NOTE]
> With Junos BGP-signaled VPLS, the provider normally picks the single active PE for a multihomed site itself (BGP multihoming with `site-preference`), so the customer doesn't have to run STP towards the provider. The limitation stands either way: only one link forwards at a time.

### 4.3 Key takeaways

> [!IMPORTANT]
> VPLS creates a virtual switch but relies on inefficient data-plane MAC learning. Its limits in MAC convergence, multihoming and gateway redundancy created the need for a smarter solution: EVPN.

---

## Module 5: EVPN (The Modern Virtual Switch)

**Objectives:**
- Introduce EVPN as the next-generation L2VPN solution.
- Explain how control-plane MAC learning solves the problems of VPLS.
- List EVPN's key advantages, including active-active multihoming and distributed gateways.

### 5.1 EVPN: the next generation

**EVPN (Ethernet VPN)** does the same job as VPLS (a distributed virtual LAN) in a radically different and better way. The single biggest difference: EVPN moves MAC address learning from the **data plane** to the **control plane**.

Instead of flood-and-learn, PEs use **BGP** to advertise MAC addresses to each other. A new MAC is advertised immediately, and just as important, a MAC that is no longer reachable can be explicitly withdrawn.

### 5.2 VPLS vs. EVPN

Control-plane MAC learning is what unlocks all of EVPN's advantages over VPLS.

| Feature | VPLS | EVPN |
|---|---|---|
| **MAC learning** | Data plane (flood-and-learn) | Control plane (BGP) |
| **MAC convergence** | Slow (relies on timeouts) | Fast (BGP updates/withdrawals) |
| **Multihoming** | Active-standby (STP required) | Active-active (all links forward) |
| **Gateway redundancy** | Active-standby (VRRP required) | Active-active (distributed anycast gateway) |

### 5.3 Key takeaways

> [!IMPORTANT]
> EVPN is the modern, recommended solution for multipoint Layer 2 VPN services. By using BGP for MAC learning, it gives faster convergence, better load balancing and multihoming, and more efficient routing than VPLS. With the flavors of L2VPN understood, the next step is configuring each one, starting with the BGP-signaled pseudowire.

---

## Module 6: Exam Practice Questions

**Question 1.** Which L2VPN technology is also known as a "virtual wire" and does not perform MAC address learning?

- A) VPLS
- B) EVPN
- C) Pseudowire
- D) L3VPN

<details><summary>Answer</summary>

**C.** A pseudowire (VPWS) acts as a point-to-point virtual wire and simply carries frames from one end to the other without learning MAC addresses.

</details>

**Question 2.** What is the primary advantage of EVPN over VPLS?

- A) It is simpler to configure for point-to-point links.
- B) It uses LDP for signaling, which is more lightweight.
- C) It uses BGP for control-plane MAC address learning.
- D) It requires less bandwidth in the service provider core.

<details><summary>Answer</summary>

**C.** Moving from data-plane (flood-and-learn) to control-plane (BGP) MAC learning is the fundamental improvement that gives EVPN its advantages in scalability, convergence and multihoming.

</details>

---

## Module 7: Glossary

| Term | Definition |
|---|---|
| **Pseudowire** | A point-to-point L2VPN service that acts as a virtual wire across a provider network. Also known as VPWS. |
| **Attachment circuit (AC)** | The logical connection between a customer's CE device and the provider's PE router. |
| **L2VPN (in Junos)** | A pseudowire signaled with BGP (Kompella). |
| **L2Circuit (in Junos)** | A pseudowire signaled with LDP (Martini). |
| **VPLS (Virtual Private LAN Service)** | A point-to-multipoint L2VPN that emulates a virtual switch and uses data-plane MAC learning. |
| **EVPN (Ethernet VPN)** | A modern point-to-multipoint L2VPN that uses BGP for control-plane MAC learning, with better features than VPLS. |

---

## Module 8: Dynamic Recall

### Part 1: Recall

**What are the two fundamental service models for Layer 2 VPNs?**

<details><summary>Answer</summary>

The **virtual wire** (a point-to-point service like a long cable, e.g. a pseudowire) and the **virtual switch** (a point-to-multipoint service like a distributed switch, e.g. VPLS/EVPN).

</details>

**What is a pseudowire, and why doesn't it need to learn MAC addresses?**

<details><summary>Answer</summary>

A pseudowire (also called VPWS) is a point-to-point L2VPN that emulates a physical wire. It doesn't learn MAC addresses because there is only one possible destination: the other end of the wire. It simply forwards every frame received on one attachment circuit to the other.

</details>

**What is the primary drawback of VPLS's MAC learning method?**

<details><summary>Answer</summary>

VPLS uses **data-plane MAC learning** (flood-and-learn). It's inefficient and slow to converge because MAC removal relies on timers, and it leads to weaker designs for multihoming (active-standby) and gateway redundancy (VRRP).

</details>

**How does EVPN solve the main problems of VPLS?**

<details><summary>Answer</summary>

EVPN moves MAC learning from the data plane to the **control plane, using BGP**. MACs are advertised and withdrawn immediately, which enables fast convergence, active-active multihoming and distributed anycast gateways.

</details>

**In Junos, what is the difference between "L2VPN" and "L2Circuit"?**

<details><summary>Answer</summary>

**L2VPN** is a BGP-signaled pseudowire (Kompella), which offers autodiscovery. **L2Circuit** is an LDP-signaled pseudowire (Martini), which requires manual endpoint configuration.

</details>

### Part 2: Fusion

> [!TIP]
> **The ghost of VPLS.**
>
> Naz stared at the network monitoring screen in horror. A customer's network was flapping wildly. "What's going on?" he asked his mentor. "Every time a virtual machine moves, half their network seems to go dark for 30 seconds!"
>
> His mentor sighed. "Ah, the ghost of VPLS. The customer is using an old **VPLS** setup. It learns MACs by **flood-and-learn**. Think of it like shouting in a library every time you move seats. It's noisy, inefficient, and everyone gets confused until the old information times out."
>
> "So for our new data center project," the mentor continued, sketching on a whiteboard, "we're using **EVPN**. Instead of shouting, EVPN uses BGP as a precise, silent directory. When a MAC moves, a BGP message is sent instantly, updating everyone's directory. No shouting, no confusion, no timeouts."
>
> Naz's eyes lit up. "So EVPN's **control-plane learning** is the fix! It's the difference between an old, chaotic library and a modern, digital one with a central, instantly updated catalog." He knew exactly what to recommend for the new project.

### Part 3: Chunk and collapse

| Chunk | One-line summary | Memory hooks |
|---|---|---|
| **Pseudowire (VPWS)** | A transparent point-to-point service that acts like a long Ethernet cable, connecting exactly two customer sites. | #VirtualWire #LongCable #PointToPoint #NoMacLearning |
| **VPLS** | A point-to-multipoint virtual switch that relies on inefficient data-plane flood-and-learn for MAC addresses. | #VirtualSwitch #FloodAndLearn #ShoutingInLibrary #ActiveStandby |
| **EVPN** | The modern virtual switch that uses BGP to advertise MAC addresses, enabling fast convergence and active-active multihoming. | #SmartSwitch #BGPforMACs #TheFix #ActiveActive |
| **Signaling** | Pseudowires are set up by a signaling protocol: BGP (L2VPN/Kompella) for scalable autodiscovery, LDP (L2Circuit/Martini) for manual simplicity. | #BGPvsLDP #Kompella #Martini #AutoVsManual |

---

← [Previous: Refresher: VPNs and MPLS](002-refresher-vpns-and-mpls.md) · [Index](../README.md) · [Next: L2VPN: BGP-Signaled Pseudowires](004-bgp-signaled-l2vpn.md) →
