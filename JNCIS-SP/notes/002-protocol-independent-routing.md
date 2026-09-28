# Junos OS: Protocol Independent Routing

**Static, Aggregate & Generated Routes**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · JNCIS-SP

---

## 1. Quick Reference

| Route type | Default preference | Next-hop behavior | Primary use |
|---|---|---|---|
| **Static** | **5** | IP address, interface, `reject`, `discard` | Explicit path control, default routes |
| **Aggregate** | **130** | `reject` (default) or `discard` | Summarization for advertisement |
| **Generated** | **130** | **Inherits the primary contributor's next hop** | Conditional default routes, forwarding summaries |

All three are configured under `[edit routing-options]`.

---

## 2. Static Routes

### What is a static route?

A manually configured path to a destination. It never changes unless you change it, which gives explicit control over traffic.

- **Hierarchy:** `[edit routing-options static]`
- **Preference:** **5**, so it beats almost every dynamic protocol.
- **Next hop must be reachable:** by default Junos does **not** resolve recursively, so the next hop should normally be directly connected.

### Configuration and next-hop types

```
[edit routing-options]
user@junos# set static route <destination-prefix> next-hop <address | interface>
```

| Next hop | Behavior |
|---|---|
| IP address | A directly connected neighbor. **Required on multi-access interfaces** like Ethernet |
| Interface name | Allowed on point-to-point interfaces |
| `reject` | Drops the packet **and sends ICMP unreachable** back to the source |
| `discard` | Drops the packet **silently** |

### Advanced options

**Qualified next hop (floating static):** a backup next hop with its own, worse preference.

```
[edit routing-options]
user@junos# set static route 198.51.100.0/24 next-hop 172.30.25.1
user@junos# set static route 198.51.100.0/24 qualified-next-hop 172.30.25.5 preference 7
```

Traffic uses `172.30.25.1` (preference 5). If it becomes unreachable, `172.30.25.5` (preference 7) takes over.

**Multiple next hops:**

```
[edit routing-options]
user@junos# set static route 192.168.1.0/24 next-hop [ 172.16.1.1 172.16.2.1 ]
```

> [!NOTE]
> Both next hops appear in the routing table, but the forwarding table only uses **one** unless you apply a load-balancing policy (`routing-options forwarding-table export` with `then load-balance per-packet`). See the Routing Policy note.

**Other useful options:**

| Option | Purpose |
|---|---|
| `resolve` | Allows recursive lookup for a next hop that isn't directly connected |
| `no-readvertise` | Prevents the route from ever being exported into a dynamic protocol, even if a policy accepts it. Ideal for management routes |
| `bfd-liveness-detection` | Fast failure detection of the next hop via BFD |

---

## 3. Aggregate Routes

### What is an aggregate route?

A **summary** that represents a group of more-specific **contributing routes**.

- **Purpose:** shrink neighbors' routing tables and hide internal flaps from external peers.
- **Activation:** active only while **at least one contributing route** is active.
- **Preference:** **130**.
- **Next hop:** **`reject` by default**. A packet inside the range with no more-specific match is dropped with ICMP unreachable. You can change it to `discard`.

### Controlling contributors with policy

```
# Only routes inside 10.1.0.0/16 may contribute
[edit policy-options policy-statement aggregate-filter]
user@junos# set term 1 from route-filter 10.1.0.0/16 orlonger
user@junos# set term 1 then accept
user@junos# set term 2 then reject

# Apply to the aggregate
[edit routing-options]
user@junos# set aggregate route 10.1.0.0/16 policy aggregate-filter
```

### Advertising the aggregate into BGP

```
[edit policy-options policy-statement export-aggregate]
user@junos# set term 1 from protocol aggregate
user@junos# set term 1 from route-filter 10.1.0.0/16 exact
user@junos# set term 1 then accept

[edit protocols bgp]
user@junos# set group external export export-aggregate
```

---

## 4. Generated Routes

### What is a generated route?

Like an aggregate, it summarizes a range and is active only while a contributor exists. The difference is the **next hop**:

| | Aggregate | Generated |
|---|---|---|
| Next hop | `reject` / `discard` | **Inherited from the primary contributing route** |
| Can forward traffic? | No, it's for advertising | **Yes** |

### Choosing the primary contributor

1. The contributor with the **lowest route preference**.
2. If tied, the contributor with the **lowest-numbered prefix**.

### Use case: conditional default route

An edge router advertises `0.0.0.0/0` into OSPF **only while** it has a specific route from its ISP via BGP. If the BGP session drops, the default is withdrawn automatically.

**Steps:**
1. A policy that matches the contributing BGP route.
2. The generated route, using that policy.
3. An export policy that matches the generated route.
4. Apply the export policy to OSPF.

```
# 1. Match the contributing BGP route
set policy-options policy-statement match-bgp-route term 1 from protocol bgp
set policy-options policy-statement match-bgp-route term 1 from route-filter 10.0.0.0/16 exact
set policy-options policy-statement match-bgp-route term 1 then accept
set policy-options policy-statement match-bgp-route term 2 then reject

# 2. Conditional generated default
set routing-options generate route 0.0.0.0/0 policy match-bgp-route

# 3. Export it ('protocol aggregate' matches BOTH aggregate and generated routes)
set policy-options policy-statement export-default-route from protocol aggregate
set policy-options policy-statement export-default-route from route-filter 0.0.0.0/0 exact
set policy-options policy-statement export-default-route then accept

# 4. Apply to OSPF
set protocols ospf export export-default-route
```

> [!IMPORTANT]
> **Exam gotcha:** there is no `protocol generate` match condition. Generated routes are matched with **`from protocol aggregate`**, and show up as `Aggregate` in `show route`.

---

## 5. Verification & Troubleshooting

```
show route
```
The whole routing table (`inet.0`). Look for `Static`, `Aggregate` and generated routes.

```
show route <PREFIX> exact detail
```
Protocol, preference, next hop and age. For aggregate and generated routes, lists the **contributing routes**.

```
show route hidden
```
Routes that aren't active, often due to an invalid next hop or a policy reject.

```
show route resolution unresolved
```
Routes whose next hop can't be resolved to an egress interface.

---

## 6. Glossary

| Term | Meaning |
|---|---|
| Active route | The route chosen (lowest preference) and installed in the forwarding table |
| Contributing route | A more-specific active route inside an aggregate/generated range that keeps it active |
| FIB / forwarding table | Streamlined table in the PFE used for high-speed forwarding |
| Next-hop resolution | Working out the egress interface and link-layer address for a next hop |
| Protocol independent routing | Routes not learned from a dynamic protocol: static, aggregate, generated |
| Qualified next hop | A backup static next hop with its own preference (floating static) |
| RIB / routing table | The full database of routes on the Routing Engine |
| Routing policy | Rules controlling route acceptance and advertisement; used to filter contributors or control redistribution |
