# Juniper Studies

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan)

My notes from working through the Juniper certification tracks. Each note is a Markdown study guide with Junos configuration examples, labs, verification output and exam tips. Everything reads directly on GitHub.

---

## 📂 Collections

| Collection | What's inside | Notes |
|---|---|---|
| [JNCIS-SP](JNCIS-SP/README.md) | Service Provider Routing & Switching, Specialist: Segment Routing, protocol-independent routing, routing policy, OSPF, IS-IS, BGP | 7 |
| [Junos MPLS Fundamentals](Junos-MPLS-Fundamentals/README.md) | MPLS and RSVP-TE on Junos: labels, static LSPs, RSVP signaling, the TED, bandwidth and priorities, CSPF and admin groups, path protection, fast reroute, optimization, make-before-break | 16 + 9 recall guides |

> [!TIP]
> Suggested order: JNCIS-SP routing policy and IGPs first, then BGP, then Junos MPLS Fundamentals, and finally Segment Routing as the modern alternative to RSVP-TE.

---

## 🗂️ Repository layout

```
Juniper-Studies/
├── README.md                      ← you are here
├── JNCIS-SP/
│   ├── README.md                  ← contents of the JNCIS-SP notes
│   └── notes/                     ← study guides + images/
└── Junos-MPLS-Fundamentals/
    ├── README.md                  ← contents of the MPLS notes
    ├── notes/                     ← study guides 01–16 + images/
    └── recall/                    ← recall guides for self-testing
```

---

> [!NOTE]
> Personal study notes, not official Juniper material. Always verify against the Juniper documentation and your own lab.
