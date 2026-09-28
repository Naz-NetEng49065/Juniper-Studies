# MPLS Mechanics: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [MPLS Mechanics](../notes/02-mpls-mechanics.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🏷️ What are the four fields of the MPLS shim header and their respective bit lengths?**

<details><summary>Answer</summary>

**Label** (20 bits), **Traffic Class (TC)** (3 bits), **Bottom of Stack (S)** (1 bit), and **Time to Live (TTL)** (8 bits). Total: 32 bits.

</details>

**2. 🥪 Why is MPLS often referred to as a "Layer 2.5" protocol?**

<details><summary>Answer</summary>

Because its shim header is inserted between the Layer 2 (e.g., Ethernet) and Layer 3 (e.g., IP) headers, operating at a level between the two.

</details>

**3. 🤝 Explain the concept of "local significance" for MPLS labels.**

<details><summary>Answer</summary>

A label value is only meaningful on the specific link between two adjacent routers. The downstream router chooses the label and tells its upstream neighbor what to use. The label for the same LSP almost always changes at every hop.

</details>

**4. 🚦 What are the three main roles a router can have in an LSP?**

<details><summary>Answer</summary>

- **Ingress (head-end):** pushes the first label.
- **Transit (LSR):** swaps labels.
- **Egress (tail-end):** pops the final label (with PHP, the penultimate router has already popped it, so the egress receives the packet unlabeled).

</details>

**5. ✂️ What is Penultimate Hop Popping (PHP) and why is it beneficial?**

<details><summary>Answer</summary>

PHP is the default behavior where the second-to-last router in an LSP pops the label. It saves the egress router from doing two lookups (one for the label, then one for the IP header), speeding up forwarding.

</details>

**6. 🗺️ Which Junos routing table is used by a transit router for label switching, and which is used by an ingress router to resolve a BGP next-hop into an LSP?**

<details><summary>Answer</summary>

A transit router uses the **mpls.0** table (its active entries are pushed to the forwarding table as the LFIB). An ingress router uses the **inet.3** table to find the LSP that resolves the BGP next-hop.

</details>

**7. 👻 What is the difference between Implicit Null (Label 3) and Explicit Null (Label 0)?**

<details><summary>Answer</summary>

**Implicit Null (3)** is a control plane signal that tells the penultimate router to POP the label (enabling PHP); label 3 never appears in a packet. **Explicit Null (0)** is a real data plane label that tells the penultimate router to SWAP to label 0 and forward the labeled packet, disabling PHP. Label 0 is IPv4 Explicit Null; IPv6 Explicit Null is label 2. In Junos, enable it with `set protocols mpls explicit-null` (RSVP) or `set protocols ldp explicit-null` (LDP).

</details>

**8. 🚗 Contrast the primary use cases for LDP and RSVP.**

<details><summary>Answer</summary>

**LDP** is simple, automated, and follows the IGP path, ideal for basic connectivity. **RSVP** is more complex but powerful, used for Traffic Engineering, bandwidth reservation, and creating explicit, non-default paths.

</details>

---

## Part 2: Fusion

"This makes no sense," Naz muttered, staring at the traceroute. The MPLS label was gone one hop **before** the egress router. "It's like the package is being unwrapped before it even gets to the destination."

The senior engineer chuckled. "That's not a bug, Naz, that's a feature. It's called **Penultimate Hop Popping (PHP)**. The egress router told its neighbor, 'Hey, I'm the end of the line, just send me the raw IP packet to save me a step.' It did this by sending a control plane signal: **Label 3, the Implicit Null**."

> "But now," the engineer continued, "the voice team needs end-to-end QoS. They need the TC bits in the MPLS header to arrive untouched. So we need to stop that early unwrapping. We'll configure the egress to advertise **Label 0, the Explicit Null**. That tells the penultimate router, 'Do NOT pop the label. Swap it to 0 and send it to me.' The label is now explicitly delivered, and QoS is preserved."

Naz's eyes lit up. It wasn't just about labels; it was about the *signals* the labels represented. Label 3 was a quiet instruction to pop, while Label 0 was a loud command to deliver. He finally understood the difference between a default convenience and an explicit requirement.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🥞 **Label Stack Bits** | The S-bit in the shim header tells the router if it's the bottom label in the stack, critical for popping logic. | `#StackAware` `#ShimMagic` `#BottomsUp` `#HeaderSecrets` |
| 🎯 **QoS & Explicit Null** | Use Explicit Null to preserve MPLS TC bits at the egress when QoS marking must be retained. | `#SaveTheTC` `#QoSIntact` `#LabelZeroHero` `#NoPHPPlease` |
| 🔀 **mpls.0 vs inet.3** | mpls.0 is the label-switching table (the LFIB) used for swapping labels; inet.3 is used at ingress to resolve next-hops via LSPs. | `#mpls0isFast` `#inet3isSmart` `#ControlVsData` `#SwapZone` |
| 🔐 **Local Significance** | Each MPLS label is only meaningful between two routers; no global significance exists across hops. | `#LocalOnly` `#HandoffLabels` `#RelabelEachHop` `#TrustNobody` |
| 🧩 **MPLS Header** | The 32-bit shim header contains a 20-bit Label, 3-bit TC for QoS, 1-bit S for stacking, and an 8-bit TTL for loop prevention. | `#32bitShim` `#Layer2point5` `#LabelAnatomy` `#TCforQoS` |
| 🧑‍🔧 **Router Roles & Actions** | Ingress routers Push, Transit routers Swap, and Egress routers Pop labels to move traffic along an LSP. | `#PushSwapPop` `#HeadTailMiddle` `#LSRlife` `#PEvsP` |
| 👍 **Penultimate Hop Popping (PHP)** | The second-to-last router pops the label by default for efficiency, as instructed by the egress router advertising Implicit Null (Label 3). | `#PopEarly` `#EfficiencyHack` `#ImplicitNull` `#SaveALookup` |
| 🗂️ **Junos MPLS Tables** | BGP uses inet.3 to find LSPs, while transit routers use mpls.0 (the LFIB) for high-speed label swapping. | `#inet3forLSP` `#mpls0forSwap` `#RoutePreferenceWins` `#ControlVsDataPlane` |
| 📬 **Explicit vs. Implicit Null** | Use Implicit Null (Label 3) for default PHP, but use Explicit Null (Label 0 for IPv4, Label 2 for IPv6) when you must deliver the label header to the egress router for QoS. | `#DisablePHP` `#QoSNeedsLabels` `#SignalVsPacket` `#ExplicitDelivery` |
| ⚖️ **LDP vs. RSVP** | LDP is the easy, automatic choice that follows the IGP, while RSVP is the manual, powerful choice for traffic engineering and guaranteed bandwidth. | `#SimpleVsPowerful` `#FollowIGP` `#CustomPaths` `#TrafficBoss` |

---

*Keep up the great work, Naz!* 💪

📖 [Full notes: MPLS Mechanics](../notes/02-mpls-mechanics.md) · [Index](../README.md)
