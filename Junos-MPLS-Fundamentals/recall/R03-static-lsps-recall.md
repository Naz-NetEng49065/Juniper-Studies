# Static LSPs and the Forwarding Plane: Recall Guide

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals

📖 Full notes: [Static LSPs and the Forwarding Plane](../notes/03-static-lsps-forwarding-plane.md) · [Index](../README.md)

---

## Part 1: Recall

Test yourself first, then expand each answer.

**1. 🛠️ What are the two essential steps to enable MPLS on a Junos interface?**

<details><summary>Answer</summary>

- **Step 1 (data plane):** enable the MPLS address family on the logical interface:

  ```
  set interfaces <int> unit <unit> family mpls
  ```

- **Step 2 (control plane):** add the interface to the MPLS protocol hierarchy:

  ```
  set protocols mpls interface <int.unit>
  ```

</details>

**2. 📝 What five pieces of information are required to configure an ingress static LSP?**

<details><summary>Answer</summary>

1. LSP name
2. Role (`ingress`)
3. Destination IP (`to`)
4. Immediate next-hop IP (`next-hop`)
5. Label operation (`push`) and label value

</details>

**3. 🎯 What is the sole purpose of the `inet.3` routing table in Junos?**

<details><summary>Answer</summary>

Its job is to store the egress IP addresses of LSPs, making them available for protocols like BGP to use for next-hop resolution.

</details>

**4. 🏆 Why does BGP prefer resolving a next-hop via an LSP in `inet.3` over an IGP route in `inet.0`?**

<details><summary>Answer</summary>

Because MPLS routes have a better (lower) default route preference than IGP routes: static LSP 6, RSVP 7, LDP 9, versus OSPF internal 10 and IS-IS Level 2 internal 18. When the same next-hop exists in both tables, the route with the better preference wins.

</details>

**5. 🔄 What is the name of the Label Forwarding Information Base (LFIB) in Junos, and what is its function?**

<details><summary>Answer</summary>

The **`mpls.0`** table. It's a simple lookup table used by transit routers that maps an incoming label to an operation (swap/pop), an outgoing label, and a next-hop.

</details>

**6. 🍿 What is the label operation configured on a penultimate-hop router for a static LSP?**

<details><summary>Answer</summary>

The **pop** operation. This removes the transport label before the packet reaches the final egress router, an optimization known as PHP.

</details>

**7. 🔍 What problem does the `set protocols mpls icmp-tunneling` command solve?**

<details><summary>Answer</summary>

It solves the problem of invisible hops in a traceroute across an MPLS core. It allows transit routers (which lack BGP routes) to tunnel their ICMP TTL-exceeded messages down the LSP to the egress router, which can then route them back to the source.

</details>

**8. 🧱 What is the primary drawback of using static LSPs?**

<details><summary>Answer</summary>

They are static. They do not dynamically react to network topology changes or link failures. If a link in the path fails, the LSP breaks and traffic is black-holed.

</details>

---

## Part 2: Fusion

"It's up! The static LSP is configured," Naz announced, a little too proudly. He checked the BGP route on the ingress router. "Look, the next-hop resolution is perfect. It's using our new LSP in **`inet.3`** instead of the old IS-IS route. The route preference of 6 beat 18, just like the book said."

The senior engineer nodded. "Good. Now run a traceroute." Naz typed, and his face fell. The trace showed the first hop, then a series of asterisks, and then the final destination. "What gives? The pings work, but the trace is blind in the middle!"

> "That's the classic static LSP problem," the engineer explained. "The transit routers are just swapping labels in the **`mpls.0`** table. When your traceroute's TTL expires on a transit router, it needs to send an ICMP message back. But it's a P router in a BGP-free core, it has no BGP route to your source IP. The message gets dropped." He smiled. "Now, add **`icmp-tunneling`** to all the core routers and watch the magic happen."

Naz added the single command on each router. He ran the traceroute again. This time, every hop appeared, complete with MPLS label information. The ICMP replies were now being "tunneled" down the LSP to the egress router, which *did* have a route back. It was a simple fix for a confusing problem, and Naz finally understood how to make the invisible MPLS path visible.

---

## Part 3: Chunk & Collapse

| Chunk | Summary | Tags |
|---|---|---|
| ✅ **Enabling MPLS** | You must enable MPLS in both the data plane (`family mpls`) and the control plane (`protocols mpls`) for it to function correctly. | `#TwoStepProcess` `#DataAndControl` `#FamilyFirst` `#ProtocolToo` |
| ✍️ **Static LSP Config** | A static LSP requires manually defining every hop and label action (push, swap, pop) from ingress to egress. | `#ManualPath` `#ControlFreak` `#NoMagic` `#PushSwapPop` |
| ⭐ **inet.3 Table** | This special table is a "shortcut" list of LSP endpoints, offered to BGP as a high-priority path for next-hop resolution. | `#BGPShortcut` `#VIPPhonebook` `#RoutePreferenceWins` `#LSP-Endpoints` |
| 🚀 **mpls.0 (LFIB)** | The `mpls.0` table is the super-fast cheat-sheet for transit routers, telling them exactly how to swap an incoming label for an outgoing one. | `#LabelSwapShop` `#LFIB` `#TransitLife` `#FastForwarding` |
| 🔦 **ICMP Tunneling** | This feature makes hidden traceroute hops visible by tunneling ICMP error messages through the LSP to a router that can route them back. | `#TracerouteVision` `#UnhideTheHops` `#SeeThePath` `#ICMPMagic` |

---

*Static LSPs are a great start. Keep going!* 💪

📖 [Full notes: Static LSPs and the Forwarding Plane](../notes/03-static-lsps-forwarding-plane.md) · [Index](../README.md)
