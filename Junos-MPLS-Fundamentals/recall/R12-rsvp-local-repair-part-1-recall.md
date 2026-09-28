# RSVP Local Repair, Part 1: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP Local Repair, Part 1](../notes/12-rsvp-local-repair-part-1.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. ⏳ Why is standard head-end LSP rerouting often too slow for real-time applications?**

<details><summary>Answer</summary>

Because the total downtime includes the time for the failure notification (e.g., a `PathErr` or `ResvTear` message) to travel back to the head-end, plus the time for the head-end to calculate and signal an entirely new path. This can take hundreds of milliseconds or more.

</details>

**2. 🚑 What is Local Repair, and what is the router that performs it called?**

<details><summary>Answer</summary>

Local Repair, or Fast Reroute (FRR), is a mechanism where the router immediately upstream of a failure uses a pre-built backup path to instantly reroute traffic. This router is called the **Point of Local Repair (PLR)**.

</details>

**3. ✌️ What are the two main methods of local repair described in RFC 4090?**

<details><summary>Answer</summary>

1. **One-to-One Backup:** creates a unique backup path (a **detour**) for each individual LSP.
2. **Facility Backup:** creates a single shared backup path (a **bypass**) that can protect many LSPs.

</details>

**4. ↪️ What is a "detour" in the context of RSVP local repair?**

<details><summary>Answer</summary>

A detour is the backup LSP created by the **One-to-One backup** method. Each protected LSP gets its own detour, built by every PLR along its path.

</details>

**5. 👨‍💻 Which Junos command enables the One-to-One backup method for an LSP?**

<details><summary>Answer</summary>

The `fast-reroute` statement, configured on the ingress under the specific LSP:

```
set protocols mpls label-switched-path <name> fast-reroute
```

</details>

**6. 📈 What is the primary scalability issue with the One-to-One backup method?**

<details><summary>Answer</summary>

It scales poorly. Since each LSP gets its own detour at every hop, backup RSVP state grows with (number of LSPs × number of hops). In large full-mesh designs, where the number of LSPs itself grows roughly with the square of the number of PEs, this adds up to a lot of state.

> [!NOTE]
> The original answer said the state grows "exponentially". It grows multiplicatively (LSPs × hops), which is still a serious scaling problem but not exponential.

</details>

**7. 🔄 How does the "detour merging" feature help reduce state in the One-to-One backup method?**

<details><summary>Answer</summary>

If multiple detours for the same protected LSP (or a detour and the protected LSP itself) arrive at a router and leave via the same outgoing interface and next hop, the router can merge them into a single outgoing path, reducing the number of backup LSPs it needs to maintain.

</details>

**8. 📩 What is the difference between a `ResvTear` and a `PathErr` message?**

<details><summary>Answer</summary>

A `ResvTear` is a destructive message that tears down an LSP's reservation state. A `PathErr` is a non-destructive message that simply notifies the head-end of a problem. With local repair, the PLR sends a `PathErr` ("tunnel locally repaired"), so it can keep using the repair path temporarily without the main LSP being torn down immediately.

</details>

---

## Part 2: Fusion

Alarms blared. A fiber cut between R3 and R4. Naz's screen lit up with alerts, but he stayed calm. Last time, this would have meant a full second of downtime while the head-end router (R1) learned of the failure and signaled a new path.

This time was different. "Check R3," his mentor said. Naz SSH'd to the router just upstream of the break. "R3 is the **Point of Local Repair (PLR)**," his mentor explained. "It detected the failure instantly."

> Naz ran `show mpls lsp extensive`. "The main LSP is down, but a new path is active!" he exclaimed. "It's a **detour**, just like we configured with **`fast-reroute`**. R3 didn't wait for instructions; it just switched the traffic onto its pre-built backup path."

He saw that R3 had sent a `PathErr` message back to the head-end, not a destructive `ResvTear`. The local repair was temporary, a sub-50ms patch to keep traffic flowing. While the head-end calmly calculated a new primary path, the PLR was already handling the crisis. It was the difference between a fire alarm and a fire suppression system.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| ⚡ **Local Repair (FRR)** | A mechanism that empowers the router closest to a fault (the PLR) to use a pre-built backup path for sub-50ms traffic restoration. | `#FastReroute` `#Sub50ms` `#PLR` `#LocalFix` |
| 👤 **One-to-One Backup** | This method creates a unique, dedicated backup LSP called a "detour" for every single primary LSP at each hop. | `#OneDetourPerLSP` `#FastRerouteCmd` `#PoorScaling` `#StateHeavy` |
| 🚌 **Facility Backup** | This method creates a single, shared backup LSP called a "bypass" that can protect many primary LSPs at once. | `#SharedBypass` `#LinkProtectionCmd` `#Scalable` `#Efficient` |
| 🆚 **Detour vs. Bypass** | A "detour" is for one LSP (One-to-One), while a "bypass" is for many LSPs (Facility). | `#DetourIsOne` `#BypassIsMany` `#FRR-vs-LinkProtect` `#KnowTheTerms` |

---

*Local repair is a powerful tool. Great job!* 💪

📖 [Full notes: RSVP Local Repair, Part 1](../notes/12-rsvp-local-repair-part-1.md) · [Index](../README.md)
