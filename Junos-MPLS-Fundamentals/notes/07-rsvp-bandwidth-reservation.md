# RSVP: LSP Bandwidth Reservation

**Reserving bandwidth on LSP paths and how it steers CSPF**

> Study notes by [Nazirul Roslan](https://www.linkedin.com/in/nazirulroslan) · Junos MPLS Fundamentals · Part 07

← [Previous: RSVP: The Traffic Engineering Database](06-rsvp-traffic-engineering-database.md) · [Index](../README.md) · [Next: RSVP: LSP Priorities](08-rsvp-lsp-priorities.md) →

**Contents:** [1 Concepts](#module-1-benefits-and-core-concepts) · [2 Configuration & CSPF](#module-2-configuration-and-cspf-in-action) · [3 Verification](#module-3-verification-toolkit) · [4 Advanced](#module-4-advanced-topics) · [5 Lab](#module-5-comprehensive-lab) · [6 Practice](#module-6-exam-practice-questions) · [7 Glossary](#module-7-glossary)

---

## Module 1: Benefits and Core Concepts

**Objectives:**
- Describe the problem of path congestion and how bandwidth reservation solves it.
- Explain the critical difference between a control plane reservation and a data plane policer.
- Understand that manual reservations are often a "guessing game".

### 1.1 The problem: congestion on the best path

By default, all LSPs try to take the metrically shortest path to their destination. If several high-bandwidth LSPs all use the same path, they can easily saturate the links and cause packet loss for everyone. That's frustrating when other, longer paths have plenty of spare capacity.

```
       +----+   +----+   +----+   +----+   +-----+
CE-A --| R1 |---| R2 |---| R3 |---| R4 |---| R5  |-- CE-B
AS100  +----+   +----+   +----+   +----+   +-----+   AS150
         |        |        |        |         |
       +----+   +----+   +----+   +----+   +-----+
       | R6 |---| R7 |---| R8 |---| R9 |---| R10 |
       +----+   +----+   +----+   +----+   +-----+

Problem: two 600 Mbps LSPs (R1->R5 and R6->R5) both want
the top path, which only has 1000 Mbps of capacity.
This will cause congestion.
```

### 1.2 The solution: bandwidth reservation

RSVP lets an LSP reserve a specific amount of bandwidth on every link along its path. The reservation acts as a constraint for other LSPs: if another LSP needs more bandwidth than is left on a link, it is forced to find an alternative path. Congestion is prevented before it happens.

> [!TIP]
> **Learner's perspective: booking a meeting room.** Reserving bandwidth is like booking a meeting room. Booking a room for 10 people from 2 to 3 PM doesn't physically stop an 11th person walking in. You're only updating the building's schedule (the control plane), so anyone else trying to book more than the remaining capacity at that time is told to find another room. The reservation influences future bookings; it doesn't police the room itself.

### 1.3 Control plane vs. data plane: a critical distinction

This is one of the most important concepts for the exam. **An RSVP bandwidth reservation is not a policer.**

| | Reservation behaviour |
|---|---|
| **It IS a control plane function** | The reservation is just an advertised number used by CSPF during path calculation. Its only job is to influence where **other** LSPs are placed. |
| **It is NOT a data plane policer** | By default it does not limit, shape or drop traffic. Reserve 400 Mbps but send 500 Mbps, and all 500 Mbps go down the LSP. |

The goal is to avoid congestion **proactively**, by creating path diversity from the start, rather than reactively dropping packets once congestion occurs.

### 1.4 The "guessing game" of manual reservations

In the real world LSP traffic is rarely constant. It fluctuates through the day and can spike unexpectedly (software updates, major sporting events). That makes a static, manually set reservation a bit of a guessing game.

> [!NOTE]
> **Advanced concept: auto-bandwidth.** Because manual reservations are often inaccurate, Junos offers **auto-bandwidth**. The LSP monitors its own actual traffic over time and automatically re-signals itself with a more accurate bandwidth requirement. This is beyond the JNCIS-SP scope but is a critical feature in many real-world deployments.

---

## Module 2: Configuration and CSPF in Action

**Objectives:**
- Configure a bandwidth reservation on an LSP.
- Explain how a bandwidth reservation constrains the CSPF algorithm.
- Predict the path an LSP will take based on bandwidth constraints.

### 2.1 Configuration

Adding a reservation is a one-line addition to the existing LSP configuration.

```
[edit protocols mpls]
set label-switched-path R1_TO_R5_400m to 192.168.1.5 bandwidth 400m
```

- `bandwidth 400m` requests a 400 Mbps reservation.
- Shorthand suffixes: `k` (kilobits), `m` (megabits), `g` (gigabits).

### 2.2 CSPF in action: a scenario

Using the 10-router topology above. All links are 1 Gbps.

1. **LSP 1 comes up:** an LSP from R1 to R5 is configured with `bandwidth 600m`. It takes the metrically best path (R1-R2-R3-R4-R5). Each link on this path now has only 400 Mbps available.
2. **LSP 2 is configured:** a new LSP from R1 to R4 with `bandwidth 700m`.
3. **CSPF calculation on R1:**
   - R1 needs a path to R4 that can support 700 Mbps.
   - Its TED shows that R1-R2, R2-R3 and R3-R4 only have 400 Mbps available.
   - CSPF **prunes** those links from its view of the topology because they don't meet the constraint.
   - With the top path removed, the only remaining path is the longer one via the bottom routers.

> [!IMPORTANT]
> **Result: path diversity.** The new R1-to-R4 LSP is forced onto R1-R6-R7-R8-R9-R4. The reservation stopped the two high-bandwidth LSPs from competing for the same links.

---

## Module 3: Verification Toolkit

**Objectives:**
- Use `show mpls lsp detail` to verify a configured reservation.
- Use `show rsvp interface` to view bandwidth usage on a link.
- Use `show ted database` to see how reservations affect the TED.
- Use `monitor label-switched-path` to view real-time traffic utilization.

### `show mpls lsp detail`: the reservation on the LSP itself

```
naz@R1> show mpls lsp name R1_TO_R5_400m detail
Ingress LSP: 1 sessions
192.168.1.5
  From: 192.168.1.1, State: Up, ActiveRoute: 0, LSPname: R1_TO_R5_400m
  ...
  *Primary            State: Up
    Priorities: 7 0
    Bandwidth: 400Mbps
    ...
    Computed ERO (S [L] denotes strict [loose] hops): (CSPF metric: 40)
     10.1.2.2 S 10.2.3.3 S 10.3.4.4 S 10.4.5.5 S
    ...
    Sender Tspec: rate 400Mbps peak 400Mbps m 20 M 1500
```

> [!NOTE]
> **The Sender Tspec object.** The `Bandwidth` line confirms what you configured. The **Sender Tspec** (Traffic Specification) is the part of the RSVP `Path` message that carries the bandwidth request downstream to the other routers. You'll also see the Tspec in `show rsvp session detail`.

### `show rsvp interface`: the impact on a physical link

```
naz@R3> show rsvp interface ge-0/0/0.0
RSVP interface: 1 active
                                  Active Subscr- Static      Available   Reserved  Highwater
Interface           State   resv  iption BW          BW          BW        mark
ge-0/0/0.0          Up      1     100%   1000Mbps    600Mbps     400Mbps     400Mbps
```

### `show ted database`: how the reservation is advertised to other routers

```
naz@R1> show ted database extensive R3.00 | match "To: R4|Available"
  To: R4.00(192.168.1.4), Local: 10.3.4.3, Remote: 10.3.4.4
    Available BW[priority]bps:
      [0] 600Mbps
      [1] 600Mbps
      ... (and so on for all 8 priorities)
```

### `monitor label-switched-path`: actual data plane traffic

```
naz@R1> monitor label-switched-path R1_TO_R5_400m
...
Traffic statistics:
                 Packets/second   Bytes/second
Output packets:      229              1411774
Output bytes:        [456]            [352944]
```

> [!TIP]
> **Exam tip:** this command shows the difference between the control plane reservation (400 Mbps) and the actual data plane traffic (only a few Mbps in this snapshot). Don't confuse the two!

---

## Module 4: Advanced Topics

**Objectives:**
- Describe how LSP priorities interact with bandwidth reservations.
- Explain oversubscription and undersubscription.
- Configure and verify interface subscription levels.

### 4.1 LSP priorities

You can give LSPs priorities so that more important traffic gets preferential treatment. Priorities only matter when there is **bandwidth contention**.

**Scenario:** an important LSP (for example voice) needs 400 Mbps, but the best path is already 70% used by a low-priority LSP.

- **High-priority LSP wins:** it "bumps" (preempts) the low-priority LSP off the path.
- **Low-priority LSP moves:** it's torn down from its original path and re-signaled on an alternate path with enough bandwidth.

Your most critical services always get the most optimal network resources.

> [!NOTE]
> Preemption only happens if the new LSP's **setup** priority is numerically lower (better) than the existing LSP's **hold** priority. With the Junos defaults (setup 7, hold 0) no LSP can preempt another, so you must configure priorities for this to work. See [Part 08](08-rsvp-lsp-priorities.md).

### 4.2 Interface subscription

By default RSVP can reserve up to 100% of an interface's bandwidth. You can change the advertised value:

| | What you advertise | Why |
|---|---|---|
| **Undersubscription** | **Less** than the physical bandwidth | Keep part of the link for non-RSVP traffic (plain IP routing, LDP traffic) |
| **Oversubscription** | **More** than the physical bandwidth | Common in service provider networks: statistically, not every customer uses their full bandwidth at once. A calculated risk that allows greater service density. |

### 4.3 Subscription configuration and verification

Set the subscription as a percentage, or as an absolute bandwidth value.

```
[edit protocols rsvp]
# Method 1: Use a percentage (e.g., 500% oversubscription)
set interface ge-0/0/0.0 subscription 500

# Method 2: Use an absolute value
set interface ge-0/0/2.0 bandwidth 2g
```

Verifying oversubscription on `ge-0/0/0.0` (a 1 Gbps physical link):

```
naz@R10> show rsvp interface
RSVP interfaces: 2 active
                                  Active Subscr- Static      Available   Reserved  Highwater
Interface           State   resv  iption BW          BW          BW        mark
ge-0/0/0.0          Up      0     500%   1000Mbps    5Gbps       0bps      0bps
ge-0/0/2.0          Up      0     100%   2Gbps       2Gbps       0bps      0bps
```

**Output analysis:** for `ge-0/0/0.0`, `Static BW` still reports the physical capacity (1000Mbps), but `Available BW` shows the oversubscribed value (5Gbps). The available value is what gets advertised into the TED.

---

## Module 5: Comprehensive Lab

In this lab you configure and verify RSVP bandwidth reservations and watch how they influence CSPF path selection.

### Lab topology

![8-router RSVP lab: R1–R4 across the top, R5–R8 across the bottom, vertical links R1–R5, R2–R6, R3–R7, R4–R8; CE-100 (AS100) on R1 and CE-150 (AS150) on R4; core in AS64512](images/rsvp-lab-topology.png)

All links are 1 Gbps. IGP metrics are equal on all links.

| Addressing | Format | Example |
|---|---|---|
| Loopbacks | `192.168.1.x/32` (x = router number) | R4 is `192.168.1.4` |
| Point-to-point | `10.x.y.z/24` (x, y = the two routers, z = this router) | On R2–R3, R2 is `10.2.3.2`, R3 is `10.2.3.3` |

### Part 1: Configure the initial LSP

**Goal:** create an LSP from R1 to R4 that reserves 600 Mbps.

**Configuration (on R1):**

```
set protocols mpls label-switched-path R1_TO_R4_600m to 192.168.1.4 bandwidth 600m
```

**Verification (on R3):** R3's link to R4 now has 600 Mbps reserved.

```
naz@R3> show rsvp interface ge-0/0/0.0
...
Interface           State   resv  iption BW          BW          BW        mark
ge-0/0/0.0          Up      1     100%   1000Mbps    400Mbps     600Mbps     600Mbps
```

### Part 2: Configure a constrained LSP

**Goal:** create a second LSP from R1 to R3 that needs 700 Mbps, and watch it avoid the now-constrained top path.

**Configuration (on R1):**

```
set protocols mpls label-switched-path R1_TO_R3_700m to 192.168.1.3 bandwidth 700m
```

**Verification (on R1):** the computed ERO now follows the bottom path, R1-R5-R6-R7-R3.

```
naz@R1> show mpls lsp name R1_TO_R3_700m detail
...
  Computed ERO (S [L] denotes strict [loose] hops): (CSPF metric: 40)
   10.1.5.5 S 10.5.6.6 S 10.6.7.7 S 10.3.7.3 S
...
```

> [!NOTE]
> The original guide showed `CSPF metric: 50` here. The path has four hops, and every other output in this series uses a metric of 10 per link (four hops = 40, as in the Module 3 output), so it has been corrected to 40.

---

## Module 6: Exam Practice Questions

**Question 1.** An engineer configures an LSP with `bandwidth 500m`. Later, `monitor label-switched-path` shows the LSP forwarding 650 Mbps. Which statement is true?

- A) The configuration has failed, as the LSP is exceeding its reserved bandwidth.
- B) This is expected behavior, as the bandwidth reservation is a control plane function and does not police data plane traffic.
- C) The extra 150 Mbps of traffic will be dropped by the ingress router.
- D) The LSP will be automatically re-signaled with a 650 Mbps reservation.

<details><summary>Answer</summary>

**B.** The reservation's purpose is to influence path selection for other LSPs (a control plane function). By default it does not police the traffic sent through the LSP.

</details>

**Question 2.** A 10 Gbps interface is configured with `set protocols rsvp interface ge-0/0/0.0 subscription 200`. What does `show rsvp interface` display as `Available BW`, assuming no LSPs are active?

- A) 10 Gbps
- B) 200 Mbps
- C) 20 Gbps
- D) 8 Gbps

<details><summary>Answer</summary>

**C.** A 200% subscription on a 10 Gbps interface advertises 20 Gbps of available bandwidth to RSVP.

</details>

---

## Module 7: Glossary

| Term | Definition |
|---|---|
| **Auto-bandwidth** | An LSP monitors its own traffic and automatically re-signals itself with an accurate bandwidth reservation. |
| **Bandwidth reservation** | A control plane mechanism where an LSP requests an amount of bandwidth on its path, influencing the path selection of other LSPs. |
| **LSP priority** | A configurable attribute that lets a high-priority LSP preempt ("bump") a low-priority LSP from a constrained path. |
| **Oversubscription** | Advertising more reservable bandwidth on an interface than is physically available, based on statistical usage. |
| **Pruning** | The first step of CSPF: links that don't meet the LSP's constraints (such as bandwidth) are temporarily removed from the topology before the path is calculated. |
| **Sender Tspec** | The object in an RSVP Path message that carries the bandwidth request from the sender (ingress router). |
| **Undersubscription** | Advertising less reservable bandwidth than is physically available, usually to keep capacity for non-RSVP traffic. |

---

← [Previous: RSVP: The Traffic Engineering Database](06-rsvp-traffic-engineering-database.md) · [Index](../README.md) · [Next: RSVP: LSP Priorities](08-rsvp-lsp-priorities.md) →
