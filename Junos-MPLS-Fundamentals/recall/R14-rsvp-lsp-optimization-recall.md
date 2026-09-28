# RSVP LSP Optimization: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP LSP Optimization](../notes/14-rsvp-lsp-optimization.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🧭 Why is LSP optimization necessary if RSVP is already dynamic?**

<details><summary>Answer</summary>

By default, RSVP prefers stability. Once an LSP is established, it stays on its path for its entire lifetime unless a failure occurs. Optimization lets the LSP proactively move to a better path (e.g., shorter, less congested) as network conditions change.

</details>

**2. ⏲️ How do you enable periodic LSP optimization in Junos?**

<details><summary>Answer</summary>

With the `optimize-timer` statement, either for all LSPs under `[edit protocols mpls]` or per LSP under `[edit protocols mpls label-switched-path <name>]`. The value is in seconds; the default of 0 disables periodic optimization. Optimization only applies to CSPF-computed LSPs.

</details>

**3. 📋 What are the four conditions that must ALL be met for a standard optimization to occur?**

<details><summary>Answer</summary>

1. The new path's metric must not be higher.
2. If the metrics are equal, the new path's hop count must not be higher.
3. The new path must not cause preemption of other LSPs.
4. The new path must not worsen overall congestion (based on Available Bandwidth Ratio).

</details>

**4. ⚖️ How does the congestion check (condition 4) compare two paths?**

<details><summary>Answer</summary>

It compares the **Available Bandwidth Ratio (ABR)** of the links, not the absolute bandwidth. It sorts the links on each path from lowest ABR to highest and compares them one-to-one. Every link on the new path must have an equal or better ABR than its counterpart on the old path.

</details>

**5. 💥 What is the purpose of the `optimize-aggressive` option?**

<details><summary>Answer</summary>

It makes the optimization algorithm consider only the first condition: the IGP (CSPF) metric. It ignores the checks for hop count, preemption, and congestion, making it a useful tool for forcing an LSP to move during maintenance. It exists both as an LSP statement and as `clear mpls lsp optimize-aggressive`.

</details>

**6. 👍 Is the `clear mpls lsp ... optimize` command disruptive to traffic?**

<details><summary>Answer</summary>

No. Junos uses a graceful make-before-break process where the new path is fully signaled (and self-pinged) before traffic is switched over. Note that a plain `clear mpls lsp` without `optimize` tears the LSP down and re-signals it, which *is* disruptive.

</details>

**7. 💯 What extra condition is added to the optimization algorithm if an LSP is configured with `least-fill`?**

<details><summary>Answer</summary>

A fifth condition: the new path's available bandwidth ratio must be at least 10% better than the current path's. This prevents flapping between two paths with very similar utilization.

</details>

**8. 🛡️ Can you optimize backup paths like detours and bypasses?**

<details><summary>Answer</summary>

Yes. Detours (One-to-One) can be optimized with `set protocols rsvp fast-reroute optimize-timer <seconds>`, and bypasses (Facility) with `set protocols rsvp interface <name> link-protection optimize-timer <seconds>`.

</details>

---

## Part 2: Fusion

Naz was staring at an LSP that was stubbornly taking a long, winding path across the network, even though a new, direct link had just been brought online. "I don't get it," he said. "I enabled the **`optimize-timer`**. The new path is metrically better and has fewer hops. Why won't it move?"

His mentor pointed to the monitor. "Because you're thinking like a human, not like the router's algorithm. You're forgetting the fourth and most important check: **congestion**. The router won't move the LSP if it thinks the new path will worsen overall congestion, even if it's shorter."

> They looked at the **Available Bandwidth Ratios**. The new, shorter path had a link with only 10% available bandwidth, while the worst link on the current long path had 15%. "The algorithm reorders the links from worst to best and compares them," the mentor explained. "Since 10% is worse than 15%, the check fails. The router prioritizes stability over a slightly better metric."
>
> "So I'm stuck?" Naz asked. "Not at all," his mentor replied. "For this maintenance, we need to force it. Use the `clear mpls lsp ... optimize-aggressive` command. That tells the router to ignore everything but the metric." Naz ran the command, and the LSP immediately and gracefully moved to the shorter path. He had learned a valuable lesson: "better" has a very specific, four-step meaning to an LSP.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| ⚙️ **LSP Optimization** | A feature that allows an LSP to periodically re-run CSPF to find and gracefully move to a more optimal path. | `#StabilityVsOptimality` `#OptimizeTimer` `#MakeBeforeBreak` `#NonDisruptive` |
| 🤔 **The Four-Step Algorithm** | A new path is only "better" if it passes four strict checks: metric, hop count, no preemption, and no worsening of congestion. | `#FourChecks` `#StabilityFirst` `#BetterIsComplex` `#AllOrNothing` |
| 📊 **Congestion Check (ABR)** | This crucial check compares the Available Bandwidth Ratio (percentage) of links, sorted from worst to best, to prevent moving to a more congested path. | `#ItsARatio` `#NotAbsoluteBW` `#ReorderAndCompare` `#CongestionAware` |
| 🚀 **Optimize Aggressive** | The "override button" that forces an LSP to move to a new path based only on a better metric, ignoring all other safety checks. | `#ForceTheMove` `#MetricOnly` `#MaintenanceTool` `#IgnoreSafety` |

---

*You're an optimization expert now. Fantastic job!* 💪

📖 [Full notes: RSVP LSP Optimization](../notes/14-rsvp-lsp-optimization.md) · [Index](../README.md)
