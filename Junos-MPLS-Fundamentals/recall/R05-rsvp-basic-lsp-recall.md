# RSVP Basic LSP Configuration: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [RSVP: Configuring a Basic LSP](../notes/05-rsvp-basic-lsp.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🚫 What is the purpose of the `no-cspf` command when configuring a basic RSVP LSP?**

<details><summary>Answer</summary>

It disables the Constrained Shortest Path First (CSPF) algorithm, so the router does not use the Traffic Engineering Database (TED). The Path message is simply routed hop by hop along the IGP's best path (from the normal routing table built from the LSDB).

</details>

**2. ✍️ Where is a basic RSVP LSP configured, and what are the two mandatory parameters?**

<details><summary>Answer</summary>

On the ingress router under the `[edit protocols mpls]` hierarchy. The two mandatory parameters are the LSP name (`label-switched-path <name>`) and the destination IP address (`to <ip-address>`).

</details>

**3. ↔️ Describe the two-way signaling process for establishing an RSVP LSP.**

<details><summary>Answer</summary>

First, a **Path message** is sent downstream from ingress to egress to request the LSP. Then, a **Resv message** is sent upstream from egress to ingress to confirm the reservation and distribute labels back to each hop.

</details>

**4. 🔍 What is the primary command to check if an LSP is up, and what is the most useful command to see the detailed path it has taken?**

<details><summary>Answer</summary>

Primary check: `show mpls lsp`. Detailed path: `show mpls lsp extensive`, which shows the Record Route Object (RRO).

</details>

**5. 🗺️ What is the Record Route Object (RRO) and which RSVP message builds it?**

<details><summary>Answer</summary>

The RRO is a list of the IP addresses (and, in Junos, the labels) of the routers an LSP traverses. The ingress requests it in the **Path** message, and each router adds itself as the Path travels downstream. The **Resv** message also carries an RRO, recorded hop by hop on its way back upstream; that Resv RRO is what the ingress displays as the "Received RRO" in `show mpls lsp extensive`, which is why it includes the label each hop allocated.

</details>

**6. 🏓 What is MPLS Self-Ping and what does it verify?**

<details><summary>Answer</summary>

An automatic Junos feature (RFC 7746) that sends a UDP packet (port 8503) addressed to the ingress itself, down the LSP to the egress, which then routes it back via normal IP. It verifies that the **data plane** for the LSP is functional and labels are programmed correctly before traffic is moved onto it.

</details>

**7. 📦 What are the four key objects in an RSVP Path message mentioned in the notes?**

<details><summary>Answer</summary>

1. **Session Object:** identifies the LSP (egress address and tunnel ID).
2. **Session Attribute Object:** carries the LSP's name, setup/hold priorities and flags.
3. **Label Request Object:** asks the downstream router for a label.
4. **RRO:** records the path taken.

(Other Path objects include the Sender Template, Sender Tspec and ERO.)

</details>

**8. 🧱 What happens if MPLS Self-Ping (UDP port 8503) is blocked by a firewall?**

<details><summary>Answer</summary>

The ingress can't confirm the data plane of a new LSP instance. Whenever a new instance is signaled (initial setup, make-before-break for optimization, or re-signaling after a failure or fast reroute), Junos keeps self-pinging and won't move traffic to the new instance until self-ping succeeds (or its retry period expires). The result is delayed switchovers and old and new instances of the same LSP lingering side by side. Fix it by permitting UDP 8503 in the lo0 filter.

> [!NOTE]
> The original answer said "the primary LSP may still come up, but optimization and Fast Reroute will fail". The detour/bypass itself doesn't depend on self-ping; what gets stuck is any make-before-break move to a new LSP instance. Exact timeouts vary by release.

</details>

---

## Part 2: Fusion

"Okay, the LSP is configured," Naz said, typing `commit` with a satisfying click. "Now for the moment of truth." He ran `show mpls lsp` and a wave of relief washed over him as he saw the state: `Up`. A small victory, but the job wasn't done. "It's up, but how do I know it's *really* working and taking the right path through the core?"

His mentor leaned over, pointing at the screen. "The 'Up' state just means the control plane handshake is complete. Don't just look at the state, look at the story. Run it again with `extensive`." Naz did, and a detailed log scrolled by. "See that **Record Route Object (RRO)**? That's the LSP's travel diary. It's telling you exactly which routers the **`Path` message** visited on its journey from here to the egress. And see those labels next to each IP? That's the souvenir the **`Resv` message** brought back from each hop, telling us which label to use."

> "But what about the data plane?" Naz asked, pointing to a specific line. "The logs say 'Self-ping ended successfully.' What's that?"
>
> "That's Junos being smart and saving you a step," the mentor replied. "After the control plane builds the tunnel, the ingress router automatically sends a **MPLS Self-Ping**. It's a special UDP packet addressed to itself, sent all the way down the new LSP. When it gets that packet back, it knows for a fact that the labels were programmed correctly at every hop and the data plane is solid. It's your automatic proof that the tunnel is truly open for business before you even send a single customer packet."

> [!NOTE]
> Strictly, the RRO shown at the ingress arrives in the Resv message (recorded hop by hop on the way back), which is why labels appear next to each address. It still describes the same path the Path message took.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| 🛤️ **`no-cspf`** | This command forces an RSVP LSP to ignore the TED and simply follow the IGP's best path. | `#NoCSPF` `#FollowTheIGP` `#SimplePath` `#DisableTE` |
| 🤝 **Path & Resv Messages** | The Path message travels downstream to request the tunnel, while the Resv message travels upstream to distribute the labels. | `#PathDown` `#ResvUp` `#TwoWayHandshake` `#SignalingFlow` |
| 📍 **Record Route Object (RRO)** | The RRO, found in `show mpls lsp extensive`, is the LSP's travel diary, showing the exact hop-by-hop path it has taken. | `#SeeThePath` `#LSP-Diary` `#RRO-Verification` `#ExtensiveIsKey` |
| ✅ **MPLS Self-Ping** | An automatic data plane health check where the ingress router sends a UDP packet to itself through the LSP to confirm it's working. | `#DataPlaneCheck` `#PingYourself` `#UDP8503` `#AutoVerify` |

---

*Configuration is just the beginning. Keep verifying!* 💪

📖 [Full notes: RSVP: Configuring a Basic LSP](../notes/05-rsvp-basic-lsp.md) · [Index](../README.md)
