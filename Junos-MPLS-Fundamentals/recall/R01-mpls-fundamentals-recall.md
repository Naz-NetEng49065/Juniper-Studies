# MPLS Fundamentals: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [MPLS Fundamentals](../notes/01-mpls-fundamentals.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 📚 What is the fundamental difference between a router's Routing Table (RIB) and its Forwarding Table (FIB)?**

<details><summary>Answer</summary>

The **Routing Table (RIB)** is the control plane's database containing *all* routes learned from all protocols (BGP, OSPF, static, etc.). The **Forwarding Table (FIB)** is the data plane's streamlined table containing *only the best (active) path* for each destination, which is used for high-speed packet forwarding.

</details>

**2. ✈️ Which operational plane (control or data) is associated with the RIB, and which with the FIB?**

<details><summary>Answer</summary>

The RIB belongs to the **control plane** 🧠. The FIB belongs to the **data plane** ⚡.

</details>

**3. 🗑️ What is the key piece of information found in the RIB that is deliberately excluded from the FIB?**

<details><summary>Answer</summary>

The FIB excludes information about *which protocol* a route was learned from, its metric, and alternative (inactive) paths. It only contains what is needed to forward the packet: destination prefix, next-hop, and outgoing interface.

</details>

**4. ⌨️ What are the equivalent Junos and Cisco commands for viewing the RIB and the FIB?**

<details><summary>Answer</summary>

| | Junos | Cisco |
|---|---|---|
| RIB | `show route` | `show ip route` |
| FIB | `show route forwarding-table` | `show ip cef` |

</details>

**5. 🗺️ Explain the process of "recursive lookup" as it relates to BGP next-hop resolution.**

<details><summary>Answer</summary>

When a router learns a BGP route, the next-hop is often a distant router's loopback address. The router must perform a *recursive lookup*: it searches its routing table (normally for an IGP route) to find the immediate, physical next-hop required to reach that distant BGP next-hop.

</details>

**6. 🎯 In a `show route detail` output, what is the practical difference between the "Protocol next hop" and the "Next hop"?**

<details><summary>Answer</summary>

The **Protocol next hop** is the BGP next-hop address (for example, a distant loopback). The **Next hop** is the actual, directly connected neighbor and interface that the packet will be sent to, as determined by the recursive lookup.

</details>

**7. 🐢 What was the original performance bottleneck in routers that MPLS was created to solve?**

<details><summary>Answer</summary>

In early routers, Layer 3 IP lookups (longest-prefix match) were a slow, software-based process handled by the router's main CPU. As routing tables grew, this CPU-based lookup became a major performance bottleneck.

</details>

**8. 🏷️ How did the concept of a "label" and a "Label-Switched Path (LSP)" address this original problem?**

<details><summary>Answer</summary>

The solution was to perform the slow, complex IP lookup only *once* at the network edge. The ingress router attaches a simple numerical **label** to the packet. Core routers then forward the packet based only on this label, a much simpler exact-match operation. The pre-determined path this labeled packet follows is the **LSP**.

</details>

**9. 💸 What is a "BGP-Free Core", and what is its primary operational and financial advantage?**

<details><summary>Answer</summary>

A network design where only the edge routers (PEs) run BGP. The core routers (Ps) do not run BGP; they run only the IGP and MPLS and switch labels. The advantage is that core routers can be cheaper, less complex devices optimized for high-throughput label switching, reducing overall network cost and complexity.

</details>

**10. 🚦 How does MPLS enable Traffic Engineering (TE) in a way that standard IP routing cannot?**

<details><summary>Answer</summary>

Standard IP routing sends all traffic for a destination down the single "best" path. MPLS-TE lets an administrator create explicit LSPs that take different, non-default paths through the network, steering traffic based on constraints like bandwidth needs or latency requirements.

</details>

**11. 🔄 Name the three fundamental label operations in an MPLS network.**

<details><summary>Answer</summary>

**Push** (adding a label), **Swap** (replacing an incoming label with an outgoing one), and **Pop** (removing a label).

</details>

**12. 🤔 What are the two distinct meanings of the term "MPLS" that can lead to confusion?**

<details><summary>Answer</summary>

- **The technology:** the underlying transport mechanism using labels, LSPs, and signaling protocols (LDP, RSVP).
- **The service:** the commercial product sold to enterprises, typically an "MPLS L3VPN circuit".

</details>

---

## Part 2: Fusion

Naz, you're staring at the network diagram at 2 AM, fueled by coffee and the pressure of the upcoming migration. The old network is a mess. Every single router in the core is running BGP, their CPUs constantly churning through massive routing tables. It's slow, expensive, and fragile. Your manager had called it a "house of cards."

> "With the new design, we create a **BGP-Free Core**... We just slap a simple sticker on it, a **label**. That's our **LSP**... They just **swap** the sticker for the next instruction. It's fast, simple, and the guys in the middle don't need a PhD in geography to do their job."

Suddenly, it clicks for you. The elegance of it isn't just speed; it's simplicity. The core routers become dumb, fast machines just switching labels, while the intelligence is pushed to the edge. The house of cards is being replaced by a sleek, efficient superhighway.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🎟️ **Label Operations** | Push adds, Swap replaces, and Pop removes MPLS labels as packets move through the LSP. | `#LabelWhisperer` `#SwapShop` `#PopGoesPacket` `#PushToStart` |
| 🎭 **MPLS vs MPLS-VPN** | MPLS is the transport mechanism, while MPLS-VPN is the service sold to enterprises. | `#TechVsService` `#CircuitSpeak` `#DoubleSpeak` `#MPLSDuality` |
| 🛣️ **Label Protocols: LDP vs RSVP** | LDP follows IGP paths automatically, RSVP-TE allows custom paths with constraints. | `#LazyLDP` `#RouteArtist` `#LDPvsRSVP` `#TrafficBoss` |
| 📺 **Multicast with MPLS** | MPLS creates P2MP LSPs for multicast without needing PIM in the core. | `#NoPIMZone` `#MulticastShortcut` `#TVOverMPLS` `#LSPCloner` |
| 🧠 **RIB vs. FIB** | The RIB is the control plane's complete library of all possible routes, while the FIB is the data plane's high-speed cheat-sheet containing only the best route for immediate forwarding. | `#BrainVsMuscle` `#LibraryVsCheatSheet` `#AllTheRoutes` `#JustTheBest` |
| 🪆 **BGP Next-Hop Resolution** | To use a BGP route, the router must first perform a recursive lookup using the IGP to find the immediate physical path to the BGP peer's address. | `#RouteWithinARoute` `#RussianNestingDolls` `#FindTheMessengerFirst` `#IGPforBGP` |
| 🏗️ **BGP-Free Core** | A BGP-Free Core uses MPLS to let simple core (P) routers switch labels, pushing the complexity and cost of running BGP to the intelligent edge (PE) routers. | `#DumbCoreSmartEdge` `#NoBGP4P` `#LabelTaxi` `#CheaperMiddle` |
| 🚗 **MPLS Traffic Engineering** | MPLS-TE gives you the power to create custom, non-default traffic highways (LSPs) across your network, steering traffic based on specific needs rather than just the IGP's shortest path. | `#NetworkGPS` `#NotTheShortestPath` `#TrafficCop` `#VIPLane` |

---

*Happy studying, Naz! You've got this.* 💪

📖 [Full notes: MPLS Fundamentals](../notes/01-mpls-fundamentals.md) · [Index](../README.md)
