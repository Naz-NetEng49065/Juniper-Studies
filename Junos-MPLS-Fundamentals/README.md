# Junos MPLS Fundamentals

**MPLS and RSVP-TE on Junos OS, from label basics to fast reroute and make-before-break**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan)

These notes follow the MPLS portion of my JNCIS-SP studies. They start with how labels work and static LSPs, then build up RSVP-TE step by step: signaling, the traffic engineering database, bandwidth, priorities, CSPF and admin groups, failures, path protection, local repair, optimization and make-before-break. Most notes include Junos configuration, a lab on the same vMX topology, real `show` output, exam tips and practice questions.

> [!TIP]
> Work through the notes in order. Each RSVP topic builds on the one before. When a topic has a 🧠 **recall guide**, use it a day or two later to test yourself before rereading the note.

![RSVP lab topology used throughout the series: eight vMX routers (R1–R8) in AS 64512 with CE-100 and CE-150 at each end](notes/images/rsvp-lab-topology.png)

---

## 📚 Contents

### MPLS foundations

| # | Topic | What's inside | Recall |
|---|---|---|---|
| 01 | [Introduction to MPLS](notes/01-mpls-fundamentals.md) | Routing foundations, why MPLS (BGP-free core, traffic engineering, VPNs), key terms | [🧠](recall/R01-mpls-fundamentals-recall.md) |
| 02 | [The Mechanics of MPLS](notes/02-mpls-mechanics.md) | Label structure, router roles and label actions, PHP, explicit null and reserved labels, transport protocols | [🧠](recall/R02-mpls-mechanics-recall.md) |
| 03 | [Static LSPs and the Forwarding Plane](notes/03-static-lsps-forwarding-plane.md) | Hand-built LSPs, route preference, `icmp-tunneling`, full static LSP lab | [🧠](recall/R03-static-lsps-recall.md) |

### RSVP-TE

| # | Topic | What's inside | Recall |
|---|---|---|---|
| 04 | [An Introduction to RSVP](notes/04-rsvp-introduction.md) | Path and Resv messages, ERO/RRO, CSPF vs. `no-cspf` | [🧠](recall/R04-rsvp-introduction-recall.md) |
| 05 | [Configuring a Basic RSVP LSP](notes/05-rsvp-basic-lsp.md) | Minimum config, verification, RRO, self-ping | [🧠](recall/R05-rsvp-basic-lsp-recall.md) |
| 06 | [RSVP: The Traffic Engineering Database](notes/06-rsvp-traffic-engineering-database.md) | IGP TE extensions, the TED, strict and loose paths, lab | |
| 07 | [RSVP: LSP Bandwidth Reservation](notes/07-rsvp-bandwidth-reservation.md) | Bandwidth constraints, subscription, Sender Tspec, lab | |
| 08 | [RSVP: LSP Priorities](notes/08-rsvp-lsp-priorities.md) | Setup/hold priorities, preemption, strategy, lab | |
| 09 | [RSVP: CSPF, Tie-Breakers and Admin Groups](notes/09-rsvp-cspf-tie-breakers-admin-groups.md) | CSPF steps, tie-breaking, `least-fill`/`most-fill`, link colouring, lab | |
| 10 | [LSP Failures, Errors and Session Maintenance](notes/10-lsp-failures-errors-session-maintenance.md) | PathErr, ResvErr, PathTear, soft state and refresh, lab | |
| 11 | [RSVP Primary and Secondary Paths](notes/11-rsvp-primary-secondary-paths.md) | Secondary and standby paths, retry and revert timers, lab | [🧠](recall/R11-rsvp-primary-secondary-paths-recall.md) |
| 12 | [RSVP Local Repair, Part 1](notes/12-rsvp-local-repair-part-1.md) | Point of local repair, one-to-one fast reroute (detours) | [🧠](recall/R12-rsvp-local-repair-part-1-recall.md) |
| 13 | [RSVP Local Repair, Part 2](notes/13-rsvp-local-repair-part-2.md) | Facility backup, link and node protection, bypass LSPs, lab | [🧠](recall/R13-rsvp-local-repair-part-2-recall.md) |
| 14 | [RSVP LSP Optimization](notes/14-rsvp-lsp-optimization.md) | Optimize timer, reoptimization checks, manual optimization, lab | [🧠](recall/R14-rsvp-lsp-optimization-recall.md) |
| 15 | [RSVP Make-Before-Break and Adaptive](notes/15-rsvp-make-before-break-adaptive.md) | Make-before-break, double counting, Shared Explicit, policy-based LSP steering, lab | |

### Bonus lab notes

| # | Topic | What's inside |
|---|---|---|
| 16 | [Co-Routed Bidirectional LSPs](notes/16-corouted-bidirectional-lsps.md) | `corouted-bidirectional`, verification, the FRR limitation, associated LSPs with performance monitoring |

---

## 🗂️ Folder layout

```
Junos-MPLS-Fundamentals/
├── README.md          ← you are here
├── notes/             ← the study guides (01–16)
│   └── images/        ← topologies, diagrams and lab screenshots
└── recall/            ← recall guides: questions, a story and summary cards
```

## 🔗 Related

- [Segment Routing](../JNCIS-SP/notes/000-segment-routing.md) in the JNCIS-SP notes: SR-MPLS, SRv6 and SR-TE, the modern alternative to RSVP-TE.
- [JNCIS-SP notes](../JNCIS-SP/README.md): routing policy, OSPF, IS-IS and BGP.

---

> [!NOTE]
> Personal study notes, not official Juniper material. Commands and output can vary between Junos releases and platforms, so check against the Juniper documentation and your own lab.
