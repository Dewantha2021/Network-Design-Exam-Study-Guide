# Network Design Exam - Compact Study Guide

> **5 Core Topics:** Requirements | Topology | Addressing | Protocols | Testing

---

## 📋 Question 1: General Requirements, Constraints, and Health

### How to gather documentation regarding addressing and naming schemes

| What to Document | Details |
|-----------------|---------|
| **Major Devices** | IP addressing for core infrastructure |
| **Client Networks** | End-user network ranges |
| **Server Networks** | Server/application IP ranges |
| **Oddities** | Look for discontiguous subnets |

**Hospital Issues Found:**
- ❌ Inconsistent IPv4 private addressing
- ❌ No central DNS

### Explaining potential "audities" like "discing" (Addressing Oddities/Discontiguous Subnets)

**Discontiguous Subnets** = Subnets from one area separated by different area

```
Area 1 Subnets ←─→ Area 0 (192.168.49.0) ←─→ Area 2 Subnets
     [Separated by different network area]
```

### Discussing the prioritization, media, and architectural constraints specific to a rural setting

| Constraint Type | Requirements |
|----------------|--------------|
| **Budget** | Limited funding |
| **Data Sovereignty** | Health data MUST stay in Sri Lanka → LGC 2.0 hosting |
| **Rural Fiber** | Unavailable/not feasible → **Solution: Starlink** |
| **Architectural** | Power, AC, heating, ventilation, physical security (lockable doors) |

### Addressing environmental factors that might affect connectivity technologies like ADSL

**Existing ADSL Performance:**
- ~5 Mbps down / 1 Mbps up
- 🔴 High latency/jitter
- 🔴 Frequent outages

**Wireless Factors (Starlink):**
```
Reflection → Absorption → Refraction → Diffraction
```

### Describing how to assess network health by using performance matrices such as latency and availability

| Metric | Measurement | Target |
|--------|-------------|--------|
| **Availability** | MTBF & MTTR | ≥99% for critical services |
| **Response Time** | Request → Response | Users care most about this |
| **Performance** | Latency/delay | Monitor continuously |
| **Bandwidth Utilization** | Usage vs capacity | Track trends |
| **Accuracy** | Error rates | Minimize |

---

## 🏗️ Question 2: Designing the Network Topology (Chapter 5)

### Designing a new topology for the possible design that gets the starting collector

**Architecture:** Hierarchical Model + Starlink WAN Backbone

```
┌──────────────────────────────┐
│   CORE LAYER                 │
│   3 Hub Hospitals            │
│   + LGC 2.0 Cloud            │
└─────────────┬────────────────┘
              │
┌─────────────▼────────────────┐
│   DISTRIBUTION LAYER         │
│   Policy, Segmentation       │
│   Inter-VLAN Routing         │
└─────────────┬────────────────┘
              │
┌─────────────▼────────────────┐
│   ACCESS LAYER               │
│   7 Branch Hospitals         │
│   User Connections           │
└──────────────────────────────┘
```

### Explaining the key design themes—hierarchy, redundancy, and modularity—and justifying why they are suitable for the root

| Theme | Benefits | Why Suitable |
|-------|----------|--------------|
| **Hierarchy** | • Constrains broadcast domains<br>• Simplifies scaling<br>• Reduces CPU adjacencies | Essential for 10-site network |
| **Redundancy** | • High availability<br>• No single point of failure | Critical for healthcare |
| **Modularity** | • Local changes only<br>• Easy troubleshooting | Growth without disruption |

**Redundancy Implementation:**
- Dual uplinks: Starlink (primary) + LTE/5G (backup)
- Internal failover: VRRP/HSRP

### Discussing the application of the Cisco hierarchical model (core, distribution, access layer) for a 10-site setup

| Layer | Role | Optimization | 10-Site Implementation |
|-------|------|--------------|------------------------|
| **Core** | High-speed backbone | Availability + Throughput | 3 Hubs + LGC 2.0 egress |
| **Distribution** | Policy enforcement | Traffic segmentation | Inter-VLAN routing at hubs |
| **Access** | User connectivity | PoE for phones/APs | All sites (3 hubs + 7 branches) |

### Describing how to incorporate redundant approaches to avoid issues with the chain or backbone in the design

**⚠️ AVOID:** Chain and backdoor topologies

| Layer | Redundancy Method | Technology |
|-------|------------------|-----------|
| **Layer 3** | Router failover | HSRP/VRRP |
| **Layer 2** | Loop prevention | STP (Spanning-Tree Protocol) |
| **WAN** | Dual uplinks | Starlink + LTE/5G |

### Explaining the use of VLAN technologies across switches, specifically for addressing roaming for telemedicine devices

**VLAN Benefits:**
- Constrains broadcast traffic
- Segments devices administratively

**Roaming Solution:**

```
Telemedicine Device moves between locations
         ↓
Stays in VLAN 30 (same VLAN + IP subnet)
         ↓
Addressing doesn't change = Seamless roaming
```

---

## 🔢 Question 3: Design Model for Addressing and Naming (Chapter 6)

### Differentiating between public and private IP addresses

| Type | Authority | Ranges | Usage |
|------|-----------|--------|-------|
| **Public** | IANA/ISPs | Assigned by ISP | Internet-routable |
| **Private** | Self-managed | 10.0.0.0/8<br>172.16.0.0/12<br>192.168.0.0/16<br>IPv6 ULA: fd00::/8 | Internal only |

### Explaining how NAT (Network Address Translation) is used with public IP at the rural site

**NAT Process:**

```
Internal Host (10.x.x.x:5000)
         ↓
    NAT Router
         ↓
Single Public IP (port mapping)
         ↓
    Internet
```

| Aspect | Details |
|--------|---------|
| **Why Needed** | Starlink provides only 1 public IPv4 |
| **Function** | Replaces internal IP:port → public IP:unique port |
| **Security Benefit** | Internal hosts hidden from outside world |

### Designing an IP addressing scheme for the private range, including requirements for subnetting, data, and scalability

**Structure: 10.0.0.0/8**

| Component | Range | Purpose |
|-----------|-------|---------|
| **Core/Backbone** | 10.0.0.0/22 | Large aggregation space |
| **Hubs** | 10.H.0.0/24 (H=1-3) | Each hub gets /24 |
| **Branches** | 10.B.0.0/24 (B=11-17) | Each branch gets /24 |
| **Per-VLAN** | /26 or /25 | Split site /24 by VLAN |
| **IPv6 Future** | fd00::/8 ULA | Internal IPv6 ranges |

**Design Goals:**
✅ Scalability  
✅ Data Separation  
✅ Future-Proofing

### Discussing the guidelines for assigning static versus dynamic addressing

| Factor | Static | Dynamic (DHCP) |
|--------|--------|----------------|
| **Device Count** | Few | Many |
| **High Availability** | Required | Not critical |
| **Security Tracking** | Need to track | Less critical |
| **Address Changes** | Rarely | Frequently |

**Design Choice:** DHCP with **reservations** for servers + critical devices

### Developing a DNS scheme for device applications

**Implementation:**
- 🏢 Centralized authoritative DNS in LGC 2.0
- 📍 Hierarchical naming

**Naming Rules:**

| Rule | Good ✅ | Bad ❌ |
|------|---------|--------|
| Short | hub1-sw01 | hub1-core-switch-floor-2-rack-3 |
| Meaningful | telemedicine-srv | server42 |
| Unambiguous | Unique names | Duplicate/similar |
| Distinct | Clearly different | Confusing |
| Case insensitive | lowercase | MiXeD CaSe |

---

## 🔌 Question 4: Selecting Switches and Routing Protocols

### Selecting and justifying the switching and routing protocol for the hospital network to ensure reliability for the government cloud

| Routing Type | Protocol | Justification |
|-------------|----------|---------------|
| **Interior** | OSPF | • Scalable<br>• Fast convergence<br>• Hierarchical topology support |
| **Cloud Connectivity** | IPsec + BGP | • Secure tunnels to LGC 2.0<br>• Policy-based routing |

### Explaining the difference between switching (layer two bridging) and routing, and when to use each within the hospital hierarchy

| | Switching (L2) | Routing (L3) |
|-|----------------|--------------|
| **Function** | Forwards frames (MAC) | Shares reachability info |
| **Operations** | Learn, flood, filter | Route calculation |
| **Complexity** | ✅ Easier setup | ❌ More complex |
| **Purpose** | Same network | Different networks |

**Usage Rule:**
```
Use Switching → Default choice
Use Routing   → Interconnect VLANs
```

**Hospital Application:**
- **Access Layer:** Switching dominant
- **Distribution Layer:** Routing (inter-VLAN)
- **Core Layer:** Routing (WAN egress)

### Discussing STP (Spanning Tree Protocol) enhancement and WAN protocols to manage loops and traffic separation

**STP Enhancements:**

| Enhancement | Purpose |
|------------|---------|
| Rapid PVST+ / MST | Reduce convergence time |
| BPDU Guard | Prevent unauthorized switches |
| Root Guard | Protect root bridge |

**Traffic Separation:**

| Technology | Purpose |
|-----------|---------|
| 802.1Q VLAN Trunking | Separate traffic on distribution links |
| SD-WAN | Policy-based path selection + failover |

### Comparing distance vector and link state routing protocols

| Feature | Distance-Vector (RIP) | Link-State (OSPF) |
|---------|----------------------|-------------------|
| **Database** | Networks + next hop + metric | Topological graph (routers + links) |
| **Updates** | Periodic (every 30s) | Event-driven (LSAs only on change) |
| **Best For** | Simple, flat topologies | **Hierarchical topologies ⭐** |
| **Convergence** | Slower | **Faster ⭐** |
| **Hospital Choice** | ❌ | ✅ Selected |

### Justifying the use of BGP (Border Gateway Protocol) for exterior routing to the government cloud

**BGP = Between Autonomous Systems**

```
Hospital AS ←──── BGP ────→ LGC 2.0 AS
```

| Feature | Benefit |
|---------|---------|
| **Scalable** | Route aggregation |
| **Policy-Based** | ⭐ Security + control for government cloud |
| **Exterior Protocol** | Designed for AS-to-AS communication |

---

## 🧪 Question 5: Testing Your Network (Chapter 12)

### Discussing test cases

**Focus:** Complex, intricate, **risky features** influenced by trade-offs

**Hospital Test Objectives:**

| Test | Target | How to Verify |
|------|--------|---------------|
| Starlink Throughput | ≥80 Mbps | Measure actual speed |
| Failover Time | <30 sec RTO | Simulate link failure |
| OSPF Convergence | <10 seconds | Monitor route updates |

**Example Test Case:**

```
Test: Firewall blocks specific traffic
1. Send TCP SYN request
2. Expect: TCP RST (reset) from firewall
3. Result: ✅ Pass if blocked, ❌ Fail if allowed
```

### Explaining the reason for testing, verifying goals, identifying broadness, and planning for contingency

| Reason | Purpose |
|--------|---------|
| **Verify Goals** | Meets business + technical requirements |
| **Validate Selections** | Technology works as expected |
| **Identify Risks** | Find potential implementation issues |
| **Plan Contingency** | Create **fallback plan** if project fails/delays |

### Describing the components of a test plan, including objectives, test types, throughput, and regression

**Test Plan Components:**

| # | Component | Details |
|---|-----------|---------|
| 1 | **Objectives** | Specific, concrete, clear pass/fail criteria |
| 2 | **Test Types** | • Response-time<br>• **Throughput tests**<br>• Availability<br>• **Regression tests** (ensure new changes don't break existing) |
| 3 | **Resources** | Equipment + network access |
| 4 | **Scripts** | Test procedures |
| 5 | **Timeline** | Milestones + deadlines |

### Outlining a prototype system used to test risky features, such as starting failovers or WAN connections

**Prototype = Test risky components when full-scale testing impractical**

**Hospital Approach:**

```
Phase 1: Pilot Deployment
    ↓
1 Hub + 1 Branch (prototype)
    ↓
Test Risky Features:
  • Failover to secondary WAN
  • OSPF convergence
    ↓
Validate → Roll out remaining sites
```

### Discussing the use of tools, specifically mentioning "pattern traces," to simulate testing on a production network

**Testing Tools:**

| Tool Type | Example | Use Case |
|-----------|---------|----------|
| **Simulation** | Packet Tracer | Configure OSPF, VLANs, NAT |
| **Traffic Generation** | iPerf | Throughput testing |
| **Pattern Traces** | Wireshark (Protocol Analyzer) | Capture traffic, analyze latency/jitter |
| **Network Management** | Various | Monitor/manage |

**Minimize Production Disruption:**

| Guideline | Implementation |
|-----------|----------------|
| Small changes | Incremental testing |
| Short duration | **<2 minutes per test** |
| Low impact | Test during off-peak |

---

## 🎯 Quick Memory Guide

### 3-3-3 Rule

**3 Layers:**
- Core (Hubs + Cloud)
- Distribution (Policy + Inter-VLAN)
- Access (Users)

**3 Themes:**
- Hierarchy (Scalability)
- Redundancy (Availability)
- Modularity (Local changes)

**3 Key Protocols:**
- OSPF (Interior)
- BGP (Exterior to Cloud)
- STP (Loop prevention)

### MTBF + MTTR = Availability

```
High MTBF (long time between failures)
+
Low MTTR (quick repairs)
=
≥99% Availability ✅
```

### TEST Framework

- **T**argets: Set objectives (≥80 Mbps, <30s failover, <10s convergence)
- **E**nvironment: Prototype (1 hub + 1 branch pilot)
- **S**hort: <2 min tests
- **T**ools: Packet Tracer, Wireshark, iPerf

### Addressing Hierarchy

```
10.0.0.0/8 (Private Space)
    ├── 10.0.0.0/22 (Core/Backbone)
    ├── 10.H.0.0/24 (Hubs, H=1-3)
    │       └── /26 or /25 per VLAN
    └── 10.B.0.0/24 (Branches, B=11-17)
            └── /26 or /25 per VLAN
```

### Switch vs Route

```
Switch when possible ✅
    ↓
Route to interconnect VLANs
    ↓
Access: Switch dominant
Distribution: Inter-VLAN routing
Core: WAN routing
```

---

## 📊 Critical Numbers

| Metric | Target | Context |
|--------|--------|---------|
| **Availability** | ≥99% | Critical services |
| **Response Time** | <250 ms RTT | Telemedicine |
| **Throughput** | ≥80 Mbps | Starlink WAN |
| **Failover** | <30 seconds | RTO requirement |
| **OSPF Convergence** | <10 seconds | Route updates |
| **Test Duration** | <2 minutes | Production testing |
| **ADSL (old)** | ~5 Mbps down | Poor performance |

---

*📚 Study smart, not hard! Good luck! �
