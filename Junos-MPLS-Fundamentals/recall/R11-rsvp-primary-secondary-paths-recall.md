# RSVP Primary and Secondary Paths: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP Primary and Secondary Paths](../notes/11-rsvp-primary-secondary-paths.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🛡️ What is the main purpose of configuring primary and secondary paths for an RSVP LSP?**

<details><summary>Answer</summary>

To pre-define an alternative backup path, reducing the downtime caused by having to calculate and signal a new LSP from scratch after a failure.

</details>

**2. 🔄 What does it mean when an LSP "reverts to the primary path"?**

<details><summary>Answer</summary>

It does NOT mean reverting to the exact previous hop-by-hop path. It means the ingress router calculates a NEW path that meets the original **constraints** of the named primary path.

</details>

**3. ⏱️ What are the three revertive timers and their default values?**

<details><summary>Answer</summary>

1. **`retry-timer`:** how often to try re-signaling the primary (default 30 s).
2. **`revert-timer`:** how long the new primary must be stable before traffic moves back to it (default 60 s).
3. **`retry-limit`:** how many times to try before giving up (default 0, meaning unlimited). If a non-zero limit is reached, retries stop and you must clear the LSP manually.

</details>

**4. 🔥 What is the key difference between a default secondary path and a "standby" secondary path?**

<details><summary>Answer</summary>

A default secondary is "cold standby": it's only calculated and signaled *after* the primary fails. A **`standby`** secondary is "hot standby": it is pre-signaled and always `Up`, ready for immediate use.

</details>

**5. ➗ How does Junos automatically encourage path diversity for a standby secondary path?**

<details><summary>Answer</summary>

Before running CSPF for the secondary path, Junos adds a large penalty to the metric of links used by the primary path (the notes cite 8,000,000), making them very unattractive for the backup path. This encourages diversity but doesn't guarantee it: if no other path exists, the secondary can still share links with the primary. For guaranteed diversity, use explicit paths or admin groups.

</details>

**6. 📚 By default, where does a standby secondary path exist (RIB, FIB, or both)?**

<details><summary>Answer</summary>

By default, only in the routing table (RIB, as an inactive next hop of the `inet.3` route). It is NOT installed in the forwarding table (FIB) until a failover occurs.

</details>

**7. ⚙️ How can you pre-install a backup path into the forwarding table (FIB) to minimize failover delay?**

<details><summary>Answer</summary>

Create a load-balancing policy (e.g., `then load-balance per-flow`) and apply it to the forwarding table with `set routing-options forwarding-table export <policy>`. As a side effect, the backup next hops are installed into the FIB with a higher (less preferred) weight.

</details>

**8. ✍️ What is the recommended command syntax for a load-balancing policy on modern Junos versions (21.4+)?**

<details><summary>Answer</summary>

`then load-balance per-flow`, because it accurately describes the behavior. `then load-balance per-packet` still works and, for backward compatibility, produces the same per-flow hashing on current platforms. (Check your release: older releases only offer `per-packet`.)

</details>

---

## Part 2: Fusion

"Maintenance alert!" the notification blared. A core router, R3, was going down in five minutes. Naz's heart pounded. A critical LSP for a banking client ran right through it. Last time, a similar failure caused a noticeable outage while the backup path was signaled.

But this time, Naz was prepared. He had already configured a **standby secondary path**. He quickly ran `show mpls lsp extensive` and confirmed both the primary and the standby paths were `Up`. The standby was pre-signaled and ready, automatically diverse thanks to Junos adding a massive metric to the primary's links during calculation.

> "But is it in the hardware?" his manager asked, peering over his shoulder. Naz confidently typed `show route forwarding-table ... extensive`. "Yes," he said, pointing. "The backup path is already in the **FIB**. We enabled the **load-balancing policy**, so it's pre-installed with a higher weight."

As the maintenance began and R3 went offline, Naz watched the LSP status. The switchover was instantaneous. No frantic signaling, no delay. The traffic seamlessly moved to the standby path because it was already programmed into the hardware. He had not only created a backup route; he had built a truly resilient, high-speed failover mechanism.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🥇 **Primary vs. Secondary** | A primary path is the main route, while a secondary is a "cold-standby" backup that only signals after the primary fails. | `#MainVsBackup` `#ColdStandby` `#FailoverPath` `#OnePrimaryManySecondary` |
| ⚡ **Standby Secondary** | Using the `standby` keyword creates a "hot-standby" path that is pre-signaled and always up for near-instant failover. | `#HotStandby` `#AlwaysUp` `#FastFailover` `#ReduceDowntime` |
| ⏳ **Revertive Timers** | The `retry-timer` (30 s) checks for a new primary, and the `revert-timer` (60 s) waits for stability before switching traffic back. | `#TimersControl` `#Retry30s` `#Revert60s` `#FailbackLogic` |
| 🚀 **FIB Pre-installation** | Applying a `load-balance per-flow` policy to the forwarding table pre-installs weighted backup paths into the FIB for the fastest possible switchover. | `#ZeroDelay` `#FIB-Ready` `#LoadBalanceTrick` `#InstantFailover` |

---

*Resiliency is key. Well done!* 💪

📖 [Full notes: RSVP Primary and Secondary Paths](../notes/11-rsvp-primary-secondary-paths.md) · [Index](../README.md)
