# Complete OSPF Guide for Junos OS

**Deploying, tuning and troubleshooting OSPF**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · JNCIS-SP
>
> Pairs with [003 · OSPF Fundamentals](003-ospf-fundamentals.md), which covers LSA types and area types in depth.

---

## Part 1: Foundational Principles

OSPF is the backbone of many networks. As a link-state protocol it converges quickly and scales well. This guide goes from core concepts to troubleshooting on Junos, for real-world work and the JNCIS-SP exam.

### The link-state paradigm: everyone gets a map

With distance-vector protocols, routers only know what their neighbors tell them, like getting directions one turn at a time. OSPF gives **every router in an area the complete map**: the **Link-State Database (LSDB)**. Each router runs **Dijkstra's SPF** on that map independently, which avoids loops and converges fast.

> [!TIP]
> **Learner's perspective: the GPS analogy.** Picture a fleet of cars, each with its own GPS. Every car keeps broadcasting: "I'm on this street, connected to these junctions, and traffic here moves this fast (cost)." Everyone builds the same live traffic map (the LSDB), and each GPS (SPF) works out its own best route. When there's an accident (a link failure), an update goes out, everyone updates their map, and routes are recalculated almost instantly.

---

## Part 2: The Seven States of Adjacency

Before exchanging routes, two OSPF routers must form an **adjacency**. A neighbor stuck in any state before `Full` is a classic troubleshooting symptom.

| # | State | What's happening | Common pitfall |
|---|---|---|---|
| 1 | **Down** | No Hellos received on this interface | |
| 2 | **Init** | A Hello arrived, but without our Router ID: one-way so far | ⚠️ Firewalls or ACLs blocking return traffic |
| 3 | **2-Way** | Each sees its own ID in the other's Hello. DR/BDR election happens here on multi-access links | |
| 4 | **ExStart** | Primary/secondary chosen to control the DBD exchange | |
| 5 | **Exchange** | DBDs (the LSDB "table of contents") exchanged | ⚠️ **MTU mismatch** stalls routers here |
| 6 | **Loading** | LSRs request missing LSAs; LSUs answer | |
| 7 | **Full** ✅ | LSDBs synchronized; fully adjacent | |

---

## Part 3: OSPF on Junos OS

All OSPFv2 configuration lives under `[edit protocols ospf]`.

### Router ID and area enablement

Set a stable Router ID (best practice: your loopback address), then add interfaces to an area.

```
# Best practice: set the Router ID explicitly
set routing-options router-id 10.0.0.1

# Enable OSPF on an interface in the backbone
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0

# Advertise the loopback, but don't try to form adjacencies on it
set protocols ospf area 0.0.0.0 interface lo0.0 passive
```

`passive` advertises the interface's network into OSPF but sends no Hellos, so no adjacency forms on it.

### Controlling path cost

```
# Method 1: set the cost on one interface
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 metric 100

# Method 2: change the reference bandwidth
# cost = reference-bandwidth / interface speed
set protocols ospf reference-bandwidth 100g
```

The Junos default reference bandwidth is 100 Mbps, so every link of 100 Mbps or faster costs 1. Raising it (e.g. to 100g) lets 10G links automatically beat 1G links.

### Securing adjacencies

```
# Simple password (not recommended for production)
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 authentication simple-password "MySecret"

# MD5 (preferred)
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 authentication md5 1 key "MySecureKey"
```

### High availability

```
# BFD for sub-second failure detection
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 bfd-liveness-detection minimum-interval 300 multiplier 3

# Graceful restart: keep forwarding during a control-plane restart
set protocols ospf graceful-restart
```

### Controlling route propagation

**Inter-area summarization (on an ABR):** condense an area's prefixes into one advertisement.

```
[edit protocols ospf]
user@ABR# set area 0.0.0.1 area-range 10.20.0.0/16
```

**Redistribution (on an ASBR):** inject routes from other protocols with an export policy.

```
[edit policy-options]
policy-statement REDISTRIBUTE-STATIC {
    from protocol static;
    then accept;
}

[edit protocols ospf]
export REDISTRIBUTE-STATIC;
```

---

## Part 4: Advanced Concepts

### ABR vs. ASBR

| | ABR (Area Border Router) | ASBR (AS Boundary Router) |
|---|---|---|
| Function | Connects areas to Area 0; handles inter-area routing | Connects OSPF to external networks (BGP, internet) |
| Location | Interfaces in at least two areas, one being Area 0 | Any area except a stub or totally stubby area |
| LSAs generated | Type 3 (Summary), Type 4 (ASBR Summary) | Type 5 (External), or Type 7 in an NSSA |
| Scalability role | Shrinks the LSDB through summarization | Brings in external reachability via redistribution |

### LSA types

| Type | Name | Generated by | Scope | Purpose |
|---|---|---|---|---|
| 1 | Router LSA | All routers | Intra-area | A router's links, states and costs |
| 2 | Network LSA | DR | Intra-area | All routers on a multi-access segment |
| 3 | Summary LSA | ABR | Inter-area | Prefixes from one area into another |
| 4 | ASBR Summary LSA | ABR | Inter-area | Where to find an ASBR |
| 5 | AS External LSA | ASBR | AS-wide | Redistributed external routes |
| 7 | NSSA External LSA | ASBR in an NSSA | NSSA only | External routes inside an NSSA; ABR translates to Type 5 |

---

## Part 5: Monitoring & Troubleshooting

### Key `show` commands

| Command | Use it to |
|---|---|
| `show ospf neighbor` | **First command to run.** Anything other than `Full` is a problem |
| `show ospf interface extensive` | Check timers, cost, and the interface **MTU** |
| `show ospf database` | Inspect the map itself: exactly which LSAs you have |
| `show route protocol ospf` | See which OSPF routes were installed |
| `show ospf statistics` | Spot errors such as area mismatches or authentication failures |

### Advanced diagnostics: `traceoptions`

Logs detailed protocol events to a file. It has less impact than a live debug, but still turn it off when you're done.

```
[edit protocols ospf]
traceoptions {
    file ospf-log size 10m files 5;
    flag error detail;
    flag hello detail;
    flag state detail;
}
```

### Troubleshooting matrix

| Symptom | Most likely cause | Diagnostic commands | `traceoptions` flag |
|---|---|---|---|
| Stuck in **Init** | One-way link, firewall blocking OSPF | `show ospf neighbor` (on SRX: `show security flow session protocol ospf`) | `hello detail` |
| Stuck in **ExStart / Exchange** | **MTU mismatch** (most common), duplicate Router ID | `show ospf interface extensive \| match mtu` | `error`, `database-description` |
| Stuck in **2-Way** | Normal between DROthers; otherwise a DR/BDR election or priority issue | `show ospf interface`, `show ospf neighbor` | `hello`, `state` |
| **No adjacency at all** | Mismatched area ID, Hello/Dead timers, authentication, subnet mask or network type | `show ospf statistics`, `show configuration protocols ospf` | `error`, `hello detail` |

> [!NOTE]
> Two DROther routers on a LAN staying at **2-Way** with each other is **expected**, not a fault. They only go Full with the DR and BDR.

---

## Part 6: JNCIS-SP Exam Prep Corner

### Exam gotcha: interface routes vs. export policy

A router exports its direct routes with a policy. One interface is a normal OSPF interface, another is passive:

```
[edit protocols ospf]
export ADVERTISE-DIRECT;
area 0.0.0.0 {
    interface ge-0/0/1.0;          # connected to 172.29.12.0/24
    interface ge-0/0/2.0 {         # connected to 172.29.13.0/24
        passive;
    }
}
```

**Which routes does the neighbor learn, and how?**

| Prefix | Learned? | As |
|---|---|---|
| 172.29.12.0/24 | ✅ Yes | **Internal** OSPF route (`OSPF/10`) |
| 172.29.13.0/24 | ✅ Yes | **Internal** OSPF route (`OSPF/10`) |

**Why:** any interface configured under an OSPF area, passive or not, has its subnet advertised **natively** in the Router LSA, as an internal route. `passive` only stops Hellos and adjacencies on that link. The export policy is only for routes **not** already in OSPF (static, BGP, or direct routes on interfaces that aren't in OSPF), and those appear as **external** routes (`OSPF/150`).

> [!IMPORTANT]
> **Key takeaways**
> - `passive` = advertise the subnet as internal, but don't form neighbors on it.
> - `export` = redistribute routes from **outside** OSPF; they arrive as external (Type 5 / Type 7).
> - Even if an export policy also matches an OSPF interface's subnet, the neighbor prefers the internal route (preference 10) over any external copy (150).

> [!WARNING]
> **Other exam favorites**
> - Stub areas block Type 4/5, and on Junos the ABR only sends a default into a stub if you configure `stub default-metric`.
> - An MTU mismatch shows up as ExStart/Exchange, not as Init.
> - OSPF's default export policy is **reject**: without an export policy, static routes are never advertised.
