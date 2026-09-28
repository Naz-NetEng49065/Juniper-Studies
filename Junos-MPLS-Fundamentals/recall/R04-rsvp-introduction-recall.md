# RSVP Introduction: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP Introduction](../notes/04-rsvp-introduction.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. ⚡ What is the primary advantage of RSVP over static LSPs?**

<details><summary>Answer</summary>

RSVP provides a scalable, automatic, and dynamic way to create LSPs that can react to network changes, unlike static LSPs, which are rigid and manually intensive.

</details>

**2. 🗺️ What is the Traffic Engineering Database (TED) and how is it built?**

<details><summary>Answer</summary>

The TED is a special database containing rich link-state information such as available bandwidth and admin groups. It's built by enabling traffic engineering extensions in the IGP (IS-IS or OSPF), which then advertise this extra information.

</details>

**3. 🧠 What is the key difference between creating an RSVP LSP using the TED versus the LSDB?**

<details><summary>Answer</summary>

Using the **TED** (CSPF) allows constrained path calculation (e.g., based on bandwidth) and signals the full computed path in an ERO. Using the **LSDB** (`no-cspf`) simply follows the IGP's best path hop by hop, with no computed ERO (an ERO is still sent if you configure an explicit path with strict or loose hops).

</details>

**4. 📍 What is an Explicit Route Object (ERO)?**

<details><summary>Answer</summary>

An object in an RSVP Path message that lists the hops the LSP must take. With CSPF it's calculated by the ingress router and signaled downstream.

</details>

**5. ✅ What are the three mandatory steps to prepare a Junos network for RSVP?**

<details><summary>Answer</summary>

1. Enable MPLS on core interfaces (`family mpls` and `protocols mpls interface`).
2. Enable RSVP on the same interfaces (`protocols rsvp interface`).
3. Allow RSVP traffic (IP protocol 46) through any control plane (lo0) firewall filters.

</details>

**6. 🔛 Which IGP requires you to explicitly enable traffic engineering, and which enables it by default?**

<details><summary>Answer</summary>

You must explicitly enable it for **OSPF** (`set protocols ospf traffic-engineering`). It is enabled by default in **IS-IS**.

</details>

**7. 💔 What are the signs of a failed RSVP neighborship in the `show rsvp neighbor` output?**

<details><summary>Answer</summary>

A non-zero (growing) **Idle** time and a **HelloRx** (received) count of 0, or one that isn't incrementing, are clear signs of a problem.

</details>

**8. 📦 What is the purpose of Opaque Objects in RSVP's history?**

<details><summary>Answer</summary>

Opaque Objects allowed the original RSVP protocol to be extended for new purposes. RSVP transports these objects without understanding them, but MPLS-enabled routers can interpret their contents to perform tasks like building LSPs.

</details>

---

## Part 2: Fusion

"The new streaming service is launching, and the default IGP path is already getting congested," Naz's manager said, pointing to a flashing red link on the monitor. "We need a guaranteed, low-latency path for that video traffic, and we need it to avoid that link completely."

Naz nodded, feeling a surge of confidence. "I've got this." First, he checked the **Traffic Engineering Database (TED)** to confirm the available bandwidth on all the alternate links. He saw a clear, uncongested path.

> On the ingress router, he configured a new RSVP LSP. He specified the destination, reserved 500Mbps of bandwidth, and added a crucial constraint: `exclude congested-link-group`. The router instantly ran a constrained path calculation, found the best route that met all the requirements, and encoded it into an **Explicit Route Object (ERO)**.

He ran `show rsvp neighbor` and saw healthy, bidirectional hellos. Then, `show rsvp session` confirmed the LSP was up and the ERO was active. He was no longer just letting the network guess the best path; he was the traffic engineer, telling the network exactly where to go.

> [!NOTE]
> In real Junos syntax the constraint in the story is an admin group: `set protocols mpls label-switched-path <name> admin-group exclude congested-link-group`, with bandwidth set via `bandwidth 500m`.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🚗 **RSVP vs. Static LSPs** | RSVP offers dynamic, scalable, and traffic-engineered LSPs, while static LSPs are rigid, manual, and do not adapt to network failures. | `#DynamicVsStatic` `#TrafficEngineer` `#Scalability` `#NoMoreStatics` |
| 📊 **Traffic Engineering Database (TED)** | The TED is a supercharged version of the LSDB, built by IGP extensions, that contains extra link details like bandwidth for smart path calculations. | `#SmarterLSDB` `#KnowYourLinks` `#BandwidthAware` `#IGP-TE` |
| ✍️ **Constrained Path & ERO** | When you add constraints, the ingress router uses the TED to calculate a precise path and signals it downstream using an Explicit Route Object (ERO). | `#MyWayOrTheHighway` `#ERO-Path` `#ConstraintBased` `#PathCalculator` |
| 📋 **Network Prep Checklist** | To use RSVP, you must enable MPLS and RSVP on interfaces and within the protocols hierarchy, and permit IP protocol 46 in your firewall. | `#PrepTheCore` `#EnableRSVP` `#Protocol46` `#FirewallRules` |
| 🤝 **RSVP Neighbor Verification** | The `show rsvp neighbor` command is your best friend; a healthy neighbor has an idle time of 0 and incrementing, roughly matching Tx/Rx hello counts. | `#NeighborCheck` `#ZeroIsGood` `#HelloTxRx` `#Troubleshooting101` |

---

*You're mastering the protocols. Excellent work!* 💪

📖 [Full notes: RSVP Introduction](../notes/04-rsvp-introduction.md) · [Index](../README.md)
