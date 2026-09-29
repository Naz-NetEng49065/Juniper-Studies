# Juniper Studies

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan)

My notes from working through the Juniper certification tracks. Each note is a Markdown study guide with Junos configuration examples, labs, verification output and exam tips. Everything reads directly on GitHub.

---

## 📂 Collections

| Collection | What's inside | Notes |
|---|---|---|
| [JNCIS-SP](JNCIS-SP/README.md) | Service Provider Routing & Switching, Specialist: Segment Routing, protocol-independent routing, routing policy, OSPF, IS-IS, BGP | 7 |
| [Junos MPLS Fundamentals](Junos-MPLS-Fundamentals/README.md) | MPLS and RSVP-TE on Junos: labels, static LSPs, RSVP signaling, the TED, bandwidth and priorities, CSPF and admin groups, path protection, fast reroute, optimization, make-before-break | 16 + 9 recall guides |
| [JNCIP-SP](JNCIP-SP/README.md) | Service Provider Routing & Switching, Professional | |
| ↳ [Junos Layer 2 VPNs](JNCIP-SP/JunosL2-VPNs/README.md) | BGP-signaled L2VPNs (site IDs, label blocks, multihoming, route target filtering) and LDP-signaled L2Circuits | 10 |

> [!TIP]
> Suggested order: JNCIS-SP routing policy and IGPs first, then BGP, then Junos MPLS Fundamentals and Segment Routing, and then the JNCIP-SP courses, which build on MPLS and BGP.

---

## 🗂️ Repository layout

```
Juniper-Studies/
├── README.md                      ← you are here
├── JNCIS-SP/
│   ├── README.md                  ← contents of the JNCIS-SP notes
│   └── notes/                     ← study guides + images/
├── Junos-MPLS-Fundamentals/
│   ├── README.md                  ← contents of the MPLS notes
│   ├── notes/                     ← study guides 01–16 + images/
│   └── recall/                    ← recall guides for self-testing
└── JNCIP-SP/
    ├── README.md                  ← list of JNCIP-SP courses
    └── JunosL2-VPNs/
        ├── README.md              ← contents of the Layer 2 VPN notes
        └── notes/                 ← study guides 001–010 + images/
```

---

> [!NOTE]
> Personal study notes, not official Juniper material. Always verify against the Juniper documentation and your own lab.
