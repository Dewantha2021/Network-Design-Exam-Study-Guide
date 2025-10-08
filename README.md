# Network Design Exam Study Guide

> **5 Major Questions:** Network Characterization | Topology Design | Addressing | Protocol Selection | Testing

---

## 📋 Question 1: General Requirements, Constraints, and Health

### How to gather documentation regarding addressing and naming schemes

| Documentation Item | What to Include |
|-------------------|----------------|
| **Logical Structure** | Network topology, areas, zones |
| **Physical Structure** | Device locations, connections |
| **Addressing** | IP for major devices, clients, servers |
| **Naming** | DNS structure, hostnames |
| **Wiring/Media** | Cable types, fiber runs |
| **Constraints** | Budget, regulatory, physical |
| **Health** | Performance metrics, availability |

### Explaining potential "audities" like "discing" (Addressing Oddities/Discontiguous Subnets)

**Discontiguous Subnets** = subnets from one area separated by different network area

```
Area 1 Subnets ← separated by → Area 0 (192.168.49.0) ← separated by → Area 2 Subnets
```

**Hospital Network Issues:**
- ❌ Inconsistent IPv4 private addressing
- ❌ No central DNS namespace

### Discussing the prioritization, media, and architectural constraints specific to a rural setting

| Constraint Type | Details |
|----------------|---------|
| **Budget** | Limited funding |
| **Regulatory** | Health data MUST stay in Sri Lanka → LGC 2.0 hosting required |
| **Rural Fiber** | Unavailable or economically infeasible → **Solution: Starlink** |
| **Power/HVAC** | Power, AC, heating, ventilation needed |
| **Physical Security** | Lockable doors, secure spaces |
| **Space** | Room for: conduits, patch panels, racks, work areas |

**Media Types to Check:**
Single-mode fiber | Multi-mode fiber | STP/UTP copper | Coaxial | Microwave | Laser | Radio | Infra-red

### Addressing environmental factors that might affect connectivity technologies like ADSL or the "340 network"

**Wireless Environmental Factors:**
```
Reflection → Absorption → Refraction → Diffraction
```

**Existing ADSL Problems:**
- 🔴 High latency/jitter
- 🔴 Frequent outages

### Describing how to assess network health by using performance matrices such as latency and availability

| Metric | Formula/Measure | Target |
|--------|----------------|--------|
| **Availability** | MTBF & MTTR | ≥99% for critical services |
| **Performance/Latency** | Delay, RTT | <250ms RTT for telemedicine |
| **Response Time** | Request → Response time | Most important to users |
| **Bandwidth Utilization** | Usage vs capacity | Monitor trends |
| **Accuracy** | Error rates | Minimize |
| **Efficiency** | Throughput vs resources | Optimize |

---

## 🏗️ Question 2: Designing the Network Topology

### Designing a new topology for the possible design that gets the starting collector

**Architecture:** Hierarchical Model + Starlink WAN Backbone

```
┌─────────────────────────────────────┐
│  CORE LAYER                         │
│  3 Hub Hospitals + LGC 2.0          │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  DISTRIBUTION LAYER                 │
│  Regional aggregation, routing,     │
│  VLAN segmentation                  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  ACCESS LAYER                       │
│  7 Branch Hospitals + switches +    │
│  edge firewalls                     │
└─────────────────────────────────────┘
```

### Explaining the key design themes—hierarchy, redundancy, and modularity—and justifying why they are suitable for the root

| Theme | Benefits | Implementation |
|-------|----------|----------------|
| **Hierarchy** | • Reduces device workload (no excessive CPU adjacencies)<br>• Constrains broadcast domains<br>• Simplifies design<br>• Easy to scale | 3-layer model (Core/Dist/Access) |
| **Redundancy** | • High availability<br>• No single point of failure | • Dual uplinks (Starlink + LTE/5G)<br>• VRRP/HSRP failover |
| **Modularity** | • Local changes only<br>• Easy to add new sites | New additions affect only directly-connected devices |

### Discussing the application of the Cisco hierarchical model (core, distribution, access layer) for a 10-site setup

| Layer | Function | Optimization | Hospital Setup (10 sites) |
|-------|----------|-------------|---------------------------|
| **Core** | High-speed backbone | Availability + Speed | 3 Hub hospitals + LGC 2.0 |
| **Distribution** | Policy enforcement | Segmentation + Routing | Inter-VLAN routing at hubs |
| **Access** | User connectivity | Access + QoS | All 10 sites (3 hubs + 7 branches) |

### Describing how to incorporate redundant approaches to avoid issues with the chain or backbone in the design

**⚠️ AVOID:** Chains & Backdoors

**✅ REDUNDANCY METHODS:**

| Method | Purpose | Technology |
|--------|---------|-----------|
| Dual WAN Links | Failover capability | Starlink (primary) + LTE/5G (secondary) |
| Router Redundancy | Local failover | VRRP/HSRP at hubs |
| Switch Redundancy | Loop prevention | STP (Spanning-Tree Protocol) |

### Explaining the use of VLAN technologies across switches in a campus network, specifically for addressing roaming for telemedicine devices

**VLAN Benefits:**
- Emulates standard LAN
- Constrains broadcast traffic
- Separates devices administratively

**Roaming Solution:**

```
Telemedicine Device moves between locations
         ↓
Stays in VLAN 30 (same VLAN + IP subnet)
         ↓
No address change needed = Seamless roaming
```

**Design:** VLAN 30 = Telemedicine devices

---

## 🔢 Question 3: Design Model for Addressing and Naming

### Differentiating between public and private IP addresses

| Type | Managed By | Ranges | Usage |
|------|-----------|--------|-------|
| **Public** | IANA/ISPs | Assigned by ISP | Internet-routable |
| **Private** | Self-managed | 10.0.0.0/8<br>172.16.0.0/12<br>192.168.0.0/16 | Internal only |

### Explaining how NAT (Network Address Translation) is used with public IP at the ruler/rural site

**NAT Process:**

```
Internal Device (10.x.x.x:5000)
         ↓
    NAT Router
         ↓
Single Public IP (203.x.x.x:12345)
         ↓
    Internet
```

| Aspect | Details |
|--------|---------|
| **Starlink Provides** | 1 public IPv4 address |
| **NAT Location** | Edge router/provider CPE |
| **Process** | Internal IP:Port → Public IP:Unique Port |
| **Security Benefit** | Internal devices invisible from outside |

### Designing an IP addressing scheme for the private range, including requirements for subnetting, data, and scalability

**Structure: 10.0.0.0/8**

| Component | Range | Purpose |
|-----------|-------|---------|
| **Core/Backbone** | 10.0.0.0/22 | Large aggregation space |
| **Hub H (H=1-3)** | 10.H.0.0/24 | Each hub gets /24 |
| **Branch B (B=11-17)** | 10.B.0.0/24 | Each branch gets /24 |
| **Per-VLAN Subnets** | /26 or /25 | Split site /24 by VLAN |
| **IPv6 (Future)** | fd00::/8 ULA | Internal IPv6 ranges |

**Benefits:** ✅ Scalability ✅ Data Separation ✅ Future-Proofing

### Discussing the guidelines for assigning static versus dynamic addressing (e.g., static for servers, dynamic for staff devices)

| Criteria | Static | Dynamic (DHCP) |
|----------|--------|----------------|
| **Number of devices** | Few | Many |
| **Renumbering needs** | Rarely | Frequently |
| **High availability** | Required | Not critical |
| **Security tracking** | Need to track | Less critical |
| **Additional config** | Manual | DHCP provides |

**Design Choice:** DHCP with **reservations** for servers + critical devices

### Developing a DNS scheme for device applications

**Implementation:**
- 🏢 Centralized authoritative DNS in LGC 2.0
- 📍 Hierarchical naming structure

**Naming Rules:**

| Rule | Example |
|------|---------|
| Short | ✅ hub1-sw01 ❌ hub1-core-switch-floor-2-rack-3 |
| Meaningful | ✅ telemedicine-srv ❌ server42 |
| Unambiguous | ✅ unique names |
| Distinct | ✅ clearly different |
| Case insensitive | ✅ lowercase preferred |

**Format:** `hostname.site.role.gov.lk` OR `service.location.gov.lk`

---

## 🔌 Question 4: Selecting Switches and Routing Protocols

### Selecting and justifying the switching and routing protocol for the hospital network to ensure reliability for the government cloud

| Protocol Type | Protocol | Justification |
|--------------|----------|---------------|
| **Interior Routing** | OSPF | • Open standard<br>• Fast adaptation<br>• Large network support<br>• Hierarchical topology requirement |
| **Exterior/Cloud** | BGP + IPsec | • Secure tunnels to LGC 2.0<br>• Policy-based routing |

### Explaining the difference between switching (layer two bridging) and routing, and when to use each within the hospital hierarchy

| | Switching (L2) | Routing (L3) |
|-|----------------|--------------|
| **Actions** | Forward, learn MACs, flood, filter | Share reachability info |
| **Complexity** | ✅ Easier | ❌ More complex |
| **Purpose** | Same network | Different networks |
| **When to Use** | **Default choice** + VLANs | Interconnect VLANs |

**Hospital Application:**

```
Access Layer    →  Switching (dominant)
Distribution    →  Routing (inter-VLAN)
Core Layer      →  Routing (WAN egress)
```

### Discussing STP (Spanning Tree Protocol) enhancement and WAN protocols to manage loops and traffic separation

**STP Enhancements:**

| Enhancement | Purpose |
|------------|---------|
| Rapid PVST+ / MST | Reduce convergence time |
| BPDU Guard | Protect against unauthorized switches |
| Root Guard | Prevent topology changes |
| Loop Guard | Prevent forwarding loops |

**Traffic Management:**

| Technology | Purpose |
|-----------|---------|
| 802.1Q VLAN Trunking | Separate traffic on distribution links |
| SD-WAN | Intelligent path selection |
| QoS Markings | Prioritize telemedicine + EHR |

### Comparing distance vector and link state routing protocols

| Feature | Distance-Vector (RIP) | Link-State (OSPF) |
|---------|----------------------|-------------------|
| **Database** | Networks + next hop + metric | Routers + links (graph) |
| **Updates** | Periodic (every 30s) | Event-driven (LSAs) |
| **Best For** | Simple/flat/hub-spoke | **Hierarchical ⭐** |
| **Convergence** | Slower OK | **Fast required ⭐** |
| **Hospital Choice** | ❌ Not selected | ✅ **Selected** |

### Justifying the use of BGP (Border Gateway Protocol) for exterior routing to the government cloud, with consideration for policy-based decisions related to security

**BGP = Between Autonomous Systems**

```
Hospital AS ←──── BGP ────→ LGC 2.0 AS
```

| Aspect | Benefit |
|--------|---------|
| **Scalable** | Handles large routing tables |
| **Route Aggregation** | Reduces routing overhead |
| **Policy Control** | ⭐ Security policies enforcement |
| **Use Case** | Secure government cloud connection |

---

## 🧪 Question 5: Testing Your Network

### Discussing test cases

**Focus:** Complex, intricate, risky features (trade-off influenced)

**Hospital Test Objectives:**

| Test | Target | Pass/Fail |
|------|--------|-----------|
| Starlink Throughput | ≥80 Mbps | Measure actual speed |
| Failover Time (RTO) | <30 seconds | Simulate link failure |
| OSPF Convergence | <10 seconds | Check route updates |
| Security Controls | ACLs + RADIUS | Verify blocks/authentication |

**Example Test Case:**

```
1. Send traffic to blocked application
2. Check for: TCP SYN request
3. Expect: TCP RST (reset) from firewall
4. Result: ✅ Pass = RST received, ❌ Fail = Connection allowed
```

### Explaining the reason for testing, verifying goals, identifying broadness, and planning for contingency

| Reason | Objective |
|--------|-----------|
| **Verify Goals** | Meets business + technical requirements |
| **Validate Selections** | Devices + technologies work as expected |
| **Identify Bottlenecks** | Find connectivity/performance issues |
| **Plan Contingencies** | Fallback if implementation fails |
| **Acceptance Test** | Secure approval for implementation |

### Describing the components of a test plan, including objectives, test types, throughput, and regression

**Test Plan Components:**

| # | Component | Details |
|---|-----------|---------|
| 1 | **Objectives + Criteria** | Specific, concrete, clear pass/fail |
| 2 | **Test Types** | • Response-time<br>• **Throughput**<br>• Availability<br>• **Regression** (new changes don't break existing) |
| 3 | **Resources** | Equipment + network access |
| 4 | **Scripts** | Test procedures |
| 5 | **Timeline** | Milestones + deadlines |

### Outlining a prototype system used to test risky features, such as starting failovers or WAN connections

**When Full Testing Impractical → Build Prototype**

**Hospital Approach:**

```
Phase 1: Pilot Deployment
    ↓
1 Hub + 1 Branch
    ↓
Test Risky Features:
  • Failover to secondary link
  • OSPF convergence times
    ↓
Validate → Roll out to remaining sites
```

### Discussing the use of tools, specifically mentioning "pattern traces," to simulate testing on a production network while maintaining minimum disruption

**Testing Tools:**

| Tool Category | Tool | Use Case |
|--------------|------|----------|
| **Simulation** | Packet Tracer | Configure OSPF, VLANs, NAT, cloud traffic |
| **Traffic Generation** | iPerf | Throughput testing |
| **Pattern Traces** | Wireshark | Capture packets, check latency/jitter |
| **Network Management** | Various | Monitor/manage devices |
| **QoS Management** | Various | Service level monitoring |

**Minimize Production Disruption:**

| Rule | Implementation |
|------|----------------|
| Small changes | Incremental testing |
| Short duration | **<2 minutes per test** |
| Off-peak | When possible |
| Rollback plan | Always prepared |

---

## 🎯 Quick Reference Chart

| Question | Key Concepts | Critical Points |
|----------|-------------|----------------|
| **Q1: Requirements** | MTBF/MTTR, Rural constraints | ≥99% availability, <250ms RTT, Starlink |
| **Q2: Topology** | 3 Layers, 3 Themes | Core/Dist/Access, Hierarchy/Redundancy/Modularity |
| **Q3: Addressing** | 10.0.0.0/8, NAT | DHCP reservations, hierarchical DNS |
| **Q4: Protocols** | OSPF, BGP, STP | OSPF inside, BGP outside, switch first |
| **Q5: Testing** | Prototype, tools | Pilot deployment, <2min tests, regression |

---

## 📝 Memory Mnemonics

**3-3-3 Rule:**
- **3 Layers:** Core, Distribution, Access
- **3 Themes:** Hierarchy, Redundancy, Modularity  
- **3 Protocols:** OSPF (interior), BGP (exterior), STP (loops)

**FARM for Health Metrics:**
- **F**ailure (MTBF)
- **A**vailability (≥99%)
- **R**esponse time (<250ms)
- **M**TTR (repair time)

**TEST Approach:**
- **T**argets (set objectives)
- **E**nvironment (prototype)
- **S**hort tests (<2 min)
- **T**ools (Packet Tracer, Wireshark)

---

*📚 Good luck on your exam!*
