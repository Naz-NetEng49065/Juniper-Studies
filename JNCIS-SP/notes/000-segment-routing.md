# Segment Routing for JNCIS-SP

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · JNCIS-SP

**Contents:** [Fundamentals](#1-segment-routing-fundamentals) · [SR-MPLS vs. SRv6](#2-sr-mpls-vs-srv6) · [Control Plane & IGP](#3-sr-mpls-control-plane--igp-integration) · [SR-TE](#4-segment-routing-traffic-engineering-sr-te) · [BGP & SR](#5-bgp--sr-integration-for-inter-domain-routing) · [Lab](#6-lab-sr-mpls-on-junos-os) · [Practice Questions](#7-exam-practice-questions) · [Glossary](#8-glossary)

---

## 1. Segment Routing Fundamentals

**Objectives:** how SR evolved from MPLS, the idea of source routing, the segment types, and global vs. local SIDs.

### 1.1 What is Segment Routing?

Segment Routing (SR) simplifies traditional MPLS by moving path intelligence from the core to the **edge** (the head-end router). Instead of hop-by-hop signaling with LDP or RSVP, the head-end describes the path as an ordered list of instructions, called **segments**. The network just executes them in order. The core holds no per-path state, which makes SR highly scalable.

### 1.2 The MPLS limitations SR fixes

- **Multiple protocols:** even simple non-TE paths need an IGP for topology *and* LDP for labels.
- **Protocols out of sync:** IGP and LDP run independently. LDP labels are locally assigned, so each router may use a different label for the same prefix, and the IGP can find a path that LDP hasn't labelled yet, which blackholes traffic.
- **State in the core:** with LDP, every P router holds label bindings for every prefix. That limits scale and complicates troubleshooting.

```
+-----+        +-----+        +-----+
| R1  |--------| R2  |--------| R3  |
+-----+        +-----+        +-----+
  PE              P              PE

Traditional MPLS:
- IGP (OSPF/IS-IS) for IP reachability
- LDP for label distribution (separate protocol)
- Per-router state for every LSP
- Labels can differ per router for the same prefix
```

### 1.3 Segment types

A segment is simply a **forwarding instruction**.

| Group | Type | What it does |
|---|---|---|
| Service segments | Policy-driven | Tells a device *how to treat* traffic (e.g. send to an application, apply QoS) |
| Topological (IGP) | **Prefix SID / Node SID** | Tied to a prefix, usually a loopback. Follows the **IGP shortest path** |
| Topological (IGP) | **Adjacency SID** | Tied to one link. Forces traffic **out a specific interface**, overriding SPF |
| Topological (BGP) | BGP segments | Segments distributed by BGP (see section 5) |
| Special | **Binding SID** | One SID that stands for a whole pre-computed path (e.g. an SR-TE policy) |

### 1.4 Globally vs. locally significant SIDs

| Significance | Meaning | Example |
|---|---|---|
| **Global** | Same meaning on every router in the SR domain | Node SID 1004 = "go to R4" everywhere |
| **Local** | Only means something on the router that advertised it | R1's adjacency SID for its link to R2 |

> [!TIP]
> **Key takeaways**
> - SR is source routing: the ingress router defines the whole path.
> - It removes the separate label protocol and per-router LSP state.
> - SIDs are encoded as MPLS labels (SR-MPLS) or IPv6 addresses (SRv6).
> - Node SIDs are global and follow the IGP; adjacency SIDs are local and pin a specific link.

**Quick check:** which best describes a Node SID?
A) Local instruction to use a specific interface · B) Global instruction to reach a router via the IGP shortest path · C) Policy instruction for traffic handling · D) A SID representing a whole SR-TE policy

<details><summary>Answer</summary>

**B.** A globally significant instruction to reach a specific router via the IGP shortest path.
</details>

> [!WARNING]
> **Common pitfall:** a segment is not a device or a link. It's an *instruction* that can be tied to a device (Node SID) or a link (Adjacency SID).

---

## 2. SR-MPLS vs. SRv6

### 2.1 SR-MPLS: the transition strategy

SR-MPLS keeps the **MPLS data plane unchanged** and modernizes only the control plane. SIDs are encoded as MPLS labels and forwarded with normal push, swap and pop. Networks can migrate in phases while services like L3VPN keep running.

```
[R1] --- [R2] --- [R3] --- [R4]
 |        |        |        |
SID:     SID:     SID:     SID:
1001     1002     1003     1004
```

### 2.2 SRv6: the long-term vision

SRv6 removes MPLS entirely. Each SID is a full **128-bit IPv6 address**, and the segment list travels in a **Segment Routing Header (SRH)**.

- **Network programming:** a SID can encode "go to this node **and** perform this function".
- **Works across non-SRv6 routers:** the head-end puts the next SID in the IPv6 destination address, so plain IPv6 routers in between just forward it normally.

```
SRv6 Segment Routing Header (SRH)
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Next Header  |  Hdr Ext Len  | Routing Type  | Segments Left |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Last Entry   |     Flags     |              Tag              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Segment List[0] (128-bit IPv6 address)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                              ...                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Segment List[n] (128-bit IPv6 address)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
//          Optional Type Length Value objects (variable)       //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The SRH can be left out when only one segment is needed, which saves header space.

### 2.3 Coexistence and choosing between them

Juniper supports a phased move from SR-MPLS to SRv6: trial SRv6 in part of the network while SR-MPLS services keep running.

| Feature | SR-MPLS | SRv6 | Notes |
|---|---|---|---|
| SID encoding | MPLS label (20-bit) | IPv6 address (128-bit) | Affects header size and expressiveness |
| Header overhead | Low (4 bytes per label) | Higher (40-byte IPv6 header + 8-byte SRH + 16 bytes per SID) | Matters for high-performance networks |
| Network programming | Limited | Extensive | Service chaining, network slicing |
| MPLS dependency | Yes | No | SRv6 suits pure IPv6 / greenfield |
| Interoperability | MPLS-aware devices end to end | Can cross plain IPv6 segments | SRv6 is more flexible in mixed networks |

- **Choose SR-MPLS** for existing MPLS networks wanting low overhead: new control plane, same data plane.
- **Choose SRv6** for greenfield IPv6, data centers, or when network programming matters.

> [!NOTE]
> Junos SRv6 support depends on release and hardware. Check Juniper's documentation before planning a lab or deployment.

**Quick check:** the main advantage of SRv6 over SR-MPLS in a greenfield IPv6 network?
A) Lower header overhead · B) No MPLS dependency · C) Simpler IGP config · D) Guaranteed sub-50 ms failover

<details><summary>Answer</summary>

**B.** It eliminates the MPLS dependency.
</details>

> [!WARNING]
> **Common pitfalls**
> - SR-MPLS does **not** change the MPLS data plane, only the control plane.
> - SRv6 endpoints must be SRv6-aware, but transit routers only need plain IPv6.

---

## 3. SR-MPLS Control Plane & IGP Integration

### 3.1 From LDP to IGP extensions

SR puts label distribution **inside the IGP** through protocol extensions, so no LDP is needed. One protocol, fewer failure points.

```
Traditional MPLS:
+-----+    IGP    +-----+    LDP    +-----+
| R1  |-----------| R2  |-----------| R3  |
+-----+           +-----+           +-----+
       (topology)       (labels)

Segment Routing:
+-----+   IGP + SR extensions   +-----+
| R1  |-------------------------| R2  |
+-----+                         +-----+
     (topology AND labels)
```

### 3.2 OSPF extensions

OSPF carries SR information in **area-scoped Opaque LSAs (Type 10)**:

| Opaque LSA | LSID | Purpose |
|---|---|---|
| Router Information LSA | `4.x.x.x` | SR capabilities and the SRGB |
| Extended Prefix LSA | `7.x.x.x` | Prefix SIDs (Node SIDs) |
| Extended Link LSA | `8.x.x.x` | Adjacency SIDs |

### 3.3 IS-IS extensions

IS-IS simply adds new **TLVs** to its existing LSPs, which keeps config and database output tidy.

| TLV | Carries |
|---|---|
| SR-Capabilities (sub-TLV of Router Capability TLV 242) | SR capabilities and the SRGB |
| Extended IP Reachability (135 / IPv6 236) | Prefix SID sub-TLV |
| Extended IS Reachability (22) | Adjacency SID sub-TLVs (31/32) |

> [!IMPORTANT]
> **Exam tip: `wide-metrics-only`.** The SR sub-TLVs live inside the wide-metric TLVs. If you configure IS-IS SR without `set protocols isis level 2 wide-metrics-only`, the config **commits without error**, but SR information isn't advertised. It fails silently.

**Operational tip:** you can see Prefix SIDs (TLV 135) and Adjacency SIDs (TLV 22, sub-TLV 31) in a Wireshark capture of an IS-IS LSP.

### 3.4 The SR Global Block (SRGB)

The **SRGB** is a contiguous label range reserved for globally significant SIDs such as Prefix SIDs. A router advertises a Prefix SID as an **index**, and each router computes the label:

```
MPLS label = SRGB start label + index
e.g.  800000 + 4 = 800004
```

> [!TIP]
> Keep the SRGB **identical on every router**. Configure it explicitly rather than relying on the platform default; this lab uses start 800000 with range 999. Verify with `show isis overview`.

**Quick check:** in IS-IS SR, which TLV advertises a Prefix SID?
A) 22 · B) 135 · C) 31 · D) 236

<details><summary>Answer</summary>

**B. TLV 135** (for IPv4; TLV 236 carries it for IPv6).
</details>

> [!WARNING]
> **Common pitfalls**
> - **Mismatched SRGB:** routers compute different labels for the same Prefix SID and paths break.
> - **Forgetting `wide-metrics-only`** in IS-IS: silent failure.

---

## 4. Segment Routing Traffic Engineering (SR-TE)

### 4.1 RSVP-TE vs. SR-TE

| | RSVP-TE | SR-TE |
|---|---|---|
| State | Every router holds state for every LSP | **Stateless core**: the path lives in the packet |
| Signaling | Hop-by-hop RSVP | None; head-end pushes a segment list |
| Scale | Limited by per-LSP state | High |

### 4.2 Path computation models

- **Centralized:** a controller (e.g. Juniper Paragon Pathfinder) sees the whole network, computes paths against constraints (bandwidth, latency), and pushes segment lists to head-ends.
- **Distributed:** the head-end computes the path itself from the IGP link-state database.

### 4.3 Label stack limits

On Junos, the number of labels a router can push is limited per **egress interface** by `family mpls maximum-labels`. The default is typically **3**, which a long segment list plus a VPN label easily exceeds. The path is then marked unusable.

```
set interfaces ge-0/0/0 unit 0 family mpls maximum-labels 5
```

> [!IMPORTANT]
> **Exam tip:** count **all** labels: every SID in the segment list **plus** service labels such as the L3VPN label. Hardware may support fewer labels than the software allows.

### 4.4 Node SIDs vs. Adjacency SIDs for TE

| SID in the segment list | Behavior | Risk / benefit |
|---|---|---|
| **Node SID** | Reach that router via **IGP SPF** | If IGP metrics change, the path between nodes changes too |
| **Adjacency SID** | Leave via **one specific link** | Strict, metric-independent control; locally significant, popped by the advertising router |

> [!TIP]
> **Key takeaways**
> - SR-TE moves path control to the edge and keeps the core stateless.
> - Node SIDs still depend on the IGP shortest path.
> - Adjacency SIDs give strict link-by-link control.
> - Watch `maximum-labels` on Junos.

**Quick check:** a metric change makes your Node-SID-based path divert. Which SID forces a specific link regardless of metrics?
A) Node · B) Binding · C) Adjacency · D) Service

<details><summary>Answer</summary>

**C. Adjacency SID.**
</details>

> [!WARNING]
> **Common pitfalls**
> - Assuming a segment list of Node SIDs is a fixed path. It isn't.
> - Not raising `maximum-labels` on every interface that must push the full stack: the SR-TE route shows as unusable.

---

## 5. BGP & SR Integration for Inter-domain Routing

### 5.1 Why BGP?

IGP extensions distribute segments **inside** a domain. **BGP** distributes them **between** domains (autonomous systems), avoiding the manual work traditional MPLS needs for inter-domain paths.

### 5.2 BGP-LU with the Prefix-SID attribute

The foundation is **BGP Labeled Unicast (BGP-LU)**, which carries an MPLS label with each prefix. For SR it is extended with the **Prefix-SID** attribute, which carries the SID index alongside the label. The advertising router computes the label it advertises from its own SRGB plus the index.

> [!NOTE]
> **Penultimate hop popping** still applies. A router advertising a prefix it originates, to a directly connected peer, sends **implicit null (label 3)** so the peer pops the label. PHP can be disabled (e.g. `no-php`) when the egress needs to see the label.

### 5.3 Junos support

Older Junos releases (before about 20.4) did not advertise Prefix SIDs in BGP; current releases do. The standards are shared across vendors, so multi-vendor examples apply to Junos too.

**Quick check:** which BGP family carries SR-MPLS segments between domains?
A) `inet-vpn unicast` · B) `inet labeled-unicast` · C) `l3vpn unicast` · D) `ipv6 labeled-unicast`

<details><summary>Answer</summary>

**B. `inet labeled-unicast`** (for IPv4 prefixes).
</details>

> [!WARNING]
> **Common pitfalls**
> - Check your Junos release supports BGP Prefix-SID before configuring it.
> - The SR-MPLS family is `inet labeled-unicast`, not `inet-vpn`.

---

## 6. Lab: SR-MPLS on Junos OS

A hands-on walkthrough: base SR config, an explicit SR-TE path, the `maximum-labels` gotcha, and strict control with an Adjacency SID.

```
 R11 --- R1 --- R2 --- R4 --- R6 --- R12
         |     / \     /      |
         |    /   \   /       |
         |   /     \ /        |
         |  /       X         |
         | /       / \        |
         R3 ----- R5 --+------+

IP addressing: 10.x.y.z/24 (x = lower router, y = higher router, z = router ID)
R11 / R12 are CEs; R1 and R6 are PEs running an L3VPN.
```

**Prerequisites**
- Junos topology as above; MPLS and IS-IS enabled on core interfaces.
- On MX / vMX: `set chassis network-services enhanced-ip` committed and the router rebooted.
- L3VPN between PEs R1 and R6, with CEs R11 and R12.

### Part 1: IGP & SR base configuration

**Goal:** enable IS-IS SR, set a consistent SRGB, assign Node SIDs, and verify.

```
# 1. Wide metrics and SR (on R1)
set protocols isis level 2 wide-metrics-only
set protocols isis source-packet-routing

# 2. SRGB (same on every router)
set protocols isis source-packet-routing srgb start-label 800000 index-range 999

# 3. Node SID index (R1 = 1, R2 = 2 ... R6 = 6)
set protocols isis source-packet-routing node-segment ipv4-index 1

commit and-quit
```

**Verify SR is on and the SRGB allocated:**

```
naz@R1> show isis overview
  SPRING enabled: Yes
  Node segment functionality: Enabled
  Node segment ipv4 index: 1
  SRGB start label: 800000
  SRGB index range: 999
  SRGB allocation: success
```

**Verify the labels in `mpls.0`** (SRGB start + index):

```
naz@R1> show route table mpls.0 protocol isis
800002(S=0)  *[L-ISIS/10] 00:01:29, metric 2
                > to 10.1.2.2 via ge-0/0/0.12, Swap 800002
800003(S=0)  *[L-ISIS/10] 00:01:29, metric 2
                > to 10.1.3.3 via ge-0/0/0.13, Swap 800003
```

> [!TIP]
> - `wide-metrics-only` is required for the SR TLVs.
> - Use the same explicit SRGB everywhere.
> - `show isis overview` is your first verification stop.
> - Node SID label = SRGB start + index.

### Part 2: an explicit SR-TE path

**Goal:** from R1, steer traffic to R6 over a longer, non-shortest path using a segment list, and prove it with traceroute.

```
edit protocols source-packet-routing
set source-routing-path PATH-TO-R6 to 6.6.6.6 primary SL-PATH-TO-R6
set segment-list SL-PATH-TO-R6 hop hop1 ip-address 10.1.3.3
set segment-list SL-PATH-TO-R6 hop hop2 label 800005
set segment-list SL-PATH-TO-R6 hop hop3 label 800002
set segment-list SL-PATH-TO-R6 hop hop4 label 800004
set segment-list SL-PATH-TO-R6 hop hop5 label 800006
commit and-quit
```

**Verify in `inet.3`:**

```
naz@R1> show route table inet.3
6.6.6.6/32  *[SPRING-TE/10] 00:00:21, metric 3
              > via 10.1.3.3, Push 800005, Push 800002, Push 800004, Push 800006
```

**Traceroute from R11**: before, traffic took R1→R2→R4→R6; now it follows R1→R3→R5→R2→R4→R6:

```
naz@R11> traceroute 192.168.2.254 source 192.168.1.254 no-resolve
 1  192.168.1.1
 2  10.1.3.3   MPLS Label=800005 CoS=0 TTL=1 S=1
 3  10.3.5.5   MPLS Label=800002 CoS=0 TTL=1 S=1
 4  10.5.2.2   MPLS Label=800004 CoS=0 TTL=1 S=1
 5  10.2.4.4   MPLS Label=800006 CoS=0 TTL=1 S=1
 6  10.4.6.6
 7  192.168.2.254
```

### Part 3: the `maximum-labels` gotcha

A long segment list plus the VPN label exceeds the default of 3, so the SR-TE route is unusable even though the config is correct. Raise it on every interface that must push the stack:

```
edit
set groups INTERNAL-INTERFACES interfaces ge-0/0/0 unit <*> family mpls maximum-labels 5
set apply-groups INTERNAL-INTERFACES
commit and-quit
```

**Verify:**

```
naz@R1> show interfaces ge-0/0/0.13
  Family mpls:
    maximum-labels 5

naz@R1> show route table CUSTOMER.inet.0 192.168.2.0/24 extensive
192.168.2.0/24  *[BGP/170] 00:00:20, localpref 100, from 6.6.6.6
                   AS path: 200 I
                 > to 6.6.6.6 via PATH-TO-R6 (SPRING-TE)
```

### Part 4: strict control with an Adjacency SID

**Step 1: break the Node SID path.** On R5, make the R5–R2 link expensive:

```
# On R5
set protocols isis interface ge-0/0/0.25 level 2 metric 1000
commit and-quit
```

Node SID 800002 now takes the IGP's new shortest path from R5 to R2, backtracking through R3 and R1:

```
 2  10.1.3.3   MPLS Label=800005
 3  10.3.5.5   MPLS Label=800003
 4  10.5.3.3   MPLS Label=800001
 5  10.3.1.1   MPLS Label=800002
 6  10.1.2.2   MPLS Label=800004
 7  10.2.4.4   MPLS Label=800006
 8  10.4.6.6
 9  192.168.2.254
```

**Step 2: find R5's Adjacency SID toward R2:**

```
naz@R5> show route table mpls.0 protocol isis | match "to 10.5.2.2"
19(S=0)  *[L-ISIS/10] 00:00:10, metric 1
            > to 10.5.2.2 via ge-0/0/0.25, Pop
```

The Adjacency SID is **19**.

**Step 3: replace the Node SID with the Adjacency SID** on R1:

```
edit protocols source-packet-routing
delete segment-list SL-PATH-TO-R6 hop hop3
set segment-list SL-PATH-TO-R6 hop hop3 label 19
insert segment-list SL-PATH-TO-R6 hop hop3 before hop hop4
commit and-quit
```

**Verify:** traffic is back on R5→R2 directly, whatever the metric:

```
 2  10.1.3.3   MPLS Label=800005
 3  10.3.5.5   MPLS Label=19
 4  10.5.2.2   MPLS Label=800004
 5  10.2.4.4   MPLS Label=800006
 6  10.4.6.6
 7  192.168.2.254
```

> [!NOTE]
> Adjacency SIDs are locally significant and usually **dynamically allocated**, so the value can change after a reboot or flap. For production, configure static adjacency SIDs or let a controller compute paths.

### Lab checklist

- [ ] IS-IS SR configured and verified
- [ ] Consistent SRGB on every core router
- [ ] Explicit SR-TE path configured
- [ ] `maximum-labels` raised
- [ ] Path change caused by an IGP metric change observed
- [ ] Path fixed with an Adjacency SID

---

## 7. Exam Practice Questions

**Q1.** A key advantage of SR's control plane over MPLS with LDP?
A) One IGP handles topology and labels · B) Every P router keeps full LSP state · C) Dynamic bandwidth reservation · D) A separate signaling protocol builds paths

<details><summary>Answer</summary>

**A.** IGP extensions carry the segments, so no LDP is needed.
</details>

**Q2.** A metric change in the core sends traffic for a distant Node SID down an unexpected path. Why?
A) It's a Binding SID · B) Node SIDs are locally significant · C) Node SIDs follow the IGP shortest path · D) The router uses SRv6

<details><summary>Answer</summary>

**C.** Prefix/Node SIDs follow SPF, so metric changes move the path.
</details>

**Q3.** Which statement about the SRGB is correct?
A) Local label range for Adjacency SIDs · B) Must differ on every router · C) Global label range for Prefix SIDs, consistent across the domain · D) An IPv6 range for SRv6

<details><summary>Answer</summary>

**C.**
</details>

**Q4.** On MX / vMX, which command is a prerequisite for SR in this lab, and needs a reboot?
A) `set protocols mpls interface all` · B) `set protocols isis source-packet-routing` · C) `set chassis network-services enhanced-ip` · D) `set routing-options forwarding-table export SR-TE-POLICY`

<details><summary>Answer</summary>

**C.** Enhanced-IP mode must be enabled (and the router rebooted) on MX platforms for these features.
</details>

---

## 8. Glossary

| Term | Meaning |
|---|---|
| Segment | A forwarding instruction |
| SID (Segment ID) | Identifies a segment: an MPLS label (SR-MPLS) or IPv6 address (SRv6) |
| SPRING | Source Packet Routing in Networking: the head-end encodes the path as a list of SIDs |
| Node SID | Global prefix SID identifying a router; follows the IGP shortest path |
| Adjacency SID | Local SID tied to a link; forces a specific egress interface |
| Binding SID | One SID representing a whole SR-TE policy |
| SRGB | Label block for global SIDs; must be consistent across the domain |
| SR-TE | Traffic engineering with segment lists; stateless in the core |
| BGP-LU | BGP labeled unicast: carries labels with prefixes; basis for inter-domain SR |
| SRv6 | SR using IPv6 addresses as SIDs, carried in the SRH |
| Opaque LSA | OSPF LSA (Type 9/10/11) carrying non-IP information such as SR extensions |
| PHP | Penultimate hop popping: the second-to-last router pops the label |
| Wide metrics | IS-IS TLVs with larger metric fields; required for SR sub-TLVs |
