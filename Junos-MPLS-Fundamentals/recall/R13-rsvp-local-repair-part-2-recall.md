# RSVP Local Repair, Part 2: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP Local Repair, Part 2](../notes/13-rsvp-local-repair-part-2.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🚌 What is a "bypass LSP" and how does it differ from a "detour"?**

<details><summary>Answer</summary>

A **bypass LSP** is a single, shared backup path created by the Facility Backup method to protect multiple primary LSPs. A **detour** is a unique, dedicated backup path created by the One-to-One method for a single primary LSP.

</details>

**2. 📈 What is the primary advantage of Facility Backup over One-to-One Backup?**

<details><summary>Answer</summary>

**Scalability.** One bypass LSP can protect hundreds or thousands of primary LSPs, creating far less RSVP state in the network than one detour per LSP.

</details>

**3. 🥞 Explain the data plane process of label stacking when traffic enters a bypass LSP.**

<details><summary>Answer</summary>

The PLR first **swaps** the incoming label for the label the merge point expects (the label advertised by the next hop for link protection, or by the next-next-hop for node protection, learned from the RRO). It then **pushes** the outer bypass label on top. Routers along the bypass forward only on the outer label. The outer label is removed at the end of the bypass (usually by PHP at the bypass's penultimate hop), revealing the inner label, which the merge point uses to forward the packet along the original LSP.

</details>

**4. 🛡️ What is the difference between Link Protection and Node-Link Protection?**

<details><summary>Answer</summary>

**Link Protection** creates a bypass that terminates at the immediate next-hop, protecting only against the link failure. **Node-Link Protection** creates a bypass that terminates at the next-next-hop, protecting against failure of both the link and the entire downstream node.

</details>

**5. ✍️ What are the two distinct configuration steps required to enable Facility Backup in Junos?**

<details><summary>Answer</summary>

- **Step 1:** on the RSVP interfaces of every router that should act as a PLR, enable bypass creation:

  ```
  set protocols rsvp interface <name> link-protection
  ```

- **Step 2:** on the ingress LSP, request protection:

  ```
  set protocols mpls label-switched-path <name> node-link-protection
  ```

  (or `link-protection` for link-only protection).

</details>

**6. 🔍 Which command is used to view only the bypass LSPs that originate on a router?**

<details><summary>Answer</summary>

`show mpls lsp bypass`. It filters the output down to the ingress bypass LSPs on that router (named like `Bypass->10.1.2.2`), which are easy to miss otherwise. `show rsvp session` also lists them.

</details>

**7. 🧱 What is a major trade-off of Facility Backup, especially on hardware with limited capabilities?**

<details><summary>Answer</summary>

It requires pushing an additional label onto the packet. Devices that can only push a small number of labels (e.g., 3) might not be able to support complex services (like Inter-AS L3VPNs) tunneled over a bypass.

</details>

**8. 🔄 How can network topology, like a ring, impact the efficiency of Facility Backup?**

<details><summary>Answer</summary>

In a ring, a node-protecting bypass may need to send traffic past the ultimate destination to reach the next-next-hop, and then have the traffic U-turn back. This can increase latency and bandwidth usage during a failure compared to the more direct detours created by the One-to-One method.

</details>

---

## Part 2: Fusion

Naz looked at the network map, overwhelmed. "We have 500 LSPs crossing the link to R3. If we use One-to-One backup, R2 will have to create 500 unique detours. The state will be enormous!"

"Exactly," said his mentor. "That's why we use **Facility Backup**. Instead of 500 detours, R2 will create one single, shared **bypass LSP** that terminates at the next-next-hop, R4. It's the ultimate carpool lane for failed LSPs."

> "But how does R4 know which of the 500 LSPs a packet belongs to?" Naz asked.
>
> "**Label stacking**," the mentor replied, sketching on the whiteboard. "When a failure happens, R2 will swap the original LSP's label as usual, but then it will *push* a second, outer label on top, the bypass label. The routers on the bypass path only look at that outer label. When the packet gets to the end of the bypass, the outer label is popped, revealing the original inner label, which R4 can then use to forward the packet correctly. It's a tunnel within a tunnel."

Naz got it. Facility Backup wasn't just about creating a backup; it was about creating a highly scalable, efficient system that used label stacking to protect an entire "facility" (the link and the next-hop node) with minimal overhead.

> [!NOTE]
> For a bypass ending at R4 (the next-next-hop), "swap as usual" means R2 swaps to the label **R4** allocated (which it learns from the RRO), not the label R3 would have expected.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🛣️ **Facility Backup (Bypass)** | This scalable method uses a single, shared "bypass" LSP to protect many primary LSPs from a common failure. | `#SharedBypass` `#ScalableFRR` `#LinkProtection` `#OneToMany` |
| 🎯 **Link vs. Node Protection** | Link protection bypasses a failed link to the next-hop, while the superior Node protection bypasses the entire next-hop node. | `#NodeIsBetter` `#NextHopVsNextNextHop` `#FullProtection` `#Fallback` |
| 🥞 **Label Stacking** | Facility Backup works by pushing a temporary, outer bypass label on top of the original LSP's label to tunnel it around a failure. | `#TunnelInATunnel` `#PushAndPop` `#OuterLabel` `#DataPlaneMagic` |
| ⚖️ **One-to-One vs. Facility Trade-offs** | Choose Facility Backup for scalability, but consider One-to-One if your hardware has label stack limits or if you're in a ring topology. | `#ScalabilityVsSimplicity` `#LabelStackLimit` `#RingTopology` `#RightToolForTheJob` |

---

*You've mastered local repair. Amazing work!* 💪

📖 [Full notes: RSVP Local Repair, Part 2](../notes/13-rsvp-local-repair-part-2.md) · [Index](../README.md)
