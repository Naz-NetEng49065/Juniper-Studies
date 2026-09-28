# JNCIS-SP Study Notes

**Juniper Networks Certified Specialist, Service Provider Routing and Switching**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan)

These are my notes on the routing protocols and Junos features covered by the JNCIS-SP track. Each note mixes concepts, Junos `set` configuration, lab walkthroughs, verification output and exam gotchas. Everything is written for Junos OS; where Junos behaves differently from other vendors, the notes say so.

> [!TIP]
> New to the series? Start with **Protocol Independent Routing** and **Routing Policy**. Almost every other topic (OSPF, IS-IS, BGP) depends on how Junos policy works.

---

## 📚 Contents

| # | Topic | What's inside |
|---|---|---|
| 000 | [Segment Routing](notes/000-segment-routing.md) | SR-MPLS vs. SRv6, SIDs and the SRGB, OSPF/IS-IS extensions, SR-TE, 4-part lab |
| 002 | [Protocol Independent Routing](notes/002-protocol-independent-routing.md) | Static, aggregate and generated routes, conditional defaults, routing tables |
| 003 | [OSPF Fundamentals](notes/003-ospf-fundamentals.md) | Router roles, LSA types, area types, stub/NSSA behavior on Junos, lab |
| 004 | [Junos Routing Policy](notes/004-junos-routing-policy.md) | Import/export, terms and match conditions, policy chains, troubleshooting, ECMP |
| 005 | [Deploying OSPF](notes/005-deploying-ospf.md) | Adjacency states, cost, authentication, BFD, summarization, troubleshooting matrix |
| 006 | [IS-IS](notes/006-is-is.md) | Levels, NET addressing, PDUs, L1/L2 design, summarization, IPv6, authentication, lab |
| 007 | [BGP](notes/007-bgp.md) | FSM, IBGP/EBGP, attributes and path selection, route reflectors, confederations, security, labs |

---

## 🗂️ Folder layout

```
JNCIS-SP/
├── README.md                     ← you are here
├── notes/
│   ├── 000-segment-routing.md
│   ├── 002-protocol-independent-routing.md
│   ├── ...
│   ├── 007-bgp.md
│   └── images/                   ← diagrams used by the notes
└── Junos-MPLS-Fundamentals/      ← coming soon
```

## 🔜 Coming next: Junos MPLS Fundamentals

A dedicated folder for MPLS on Junos: label operations, LDP, RSVP-TE, LSP signaling and traffic engineering. It builds on the [Segment Routing](notes/000-segment-routing.md) note.

---

> [!NOTE]
> These are personal study notes, not official Juniper material. Commands and behavior can vary between Junos releases and platforms, so always check against the Juniper documentation and your own lab.
