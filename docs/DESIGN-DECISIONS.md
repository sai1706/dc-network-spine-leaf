# Design Decisions

## Architecture Choice: Spine-Leaf

### Why Spine-Leaf Over Traditional 3-Tier?

**Decision:** Use spine-leaf (Clos) architecture instead of traditional 3-tier (core-distribution-access).

**Reasoning:**
- **Predictable latency:** Maximum 2 hops between any two servers
- **No spanning tree:** BGP routing eliminates STP blocking and convergence delays
- **Horizontal scalability:** Add capacity by adding spine or leaf switches
- **ECMP load balancing:** Traffic automatically distributed across all available paths
- **Simplified operations:** Consistent configuration patterns across switch tiers

**Trade-offs:**
- Requires more spine switches than core switches in 3-tier
- Higher port count requirements on spines
- More complex initial design (but simpler operations long-term)

---

## Routing Protocol: eBGP

### Why eBGP Instead of OSPF or IS-IS?

**Decision:** Use eBGP with unique AS per switch (AS 65001-65004 for spines, 65011-65018 for leaves).

**Reasoning:**
- **Native ECMP:** BGP provides equal-cost multipath without metric tuning
- **Simplicity:** Easier to configure and troubleshoot than IGP in DC fabric
- **Scalability:** Add switches without network-wide SPF recalculations
- **Industry standard:** Proven at scale by Facebook, Google, AWS
- **Policy flexibility:** Rich policy controls via AS-path, communities, prefix-lists

**Alternative considered:** OSPF was rejected because:
- Requires careful metric tuning for ECMP
- Single failure domain (all switches in one area)
- More complex troubleshooting in large fabrics

---

## IP Addressing Scheme

### Point-to-Point Links: /31 Subnets

**Decision:** Use /31 subnets (RFC 3021) for all spine-leaf interconnects.

**Reasoning:**
- **IP conservation:** Saves 2 IPs per link vs /30 (64 IPs saved across 32 links)
- **Simplicity:** Only two addresses needed for point-to-point links
- **Industry standard:** Commonly used in modern data centers

**Example:** SPINE-1 ↔ LEAF-1 uses 10.1.1.0/31 (only .0 and .1 are usable)

### Server Subnets: /24 Per Rack

**Decision:** Allocate one /24 subnet per leaf switch.

**Reasoning:**
- **Sufficient capacity:** 254 IPs supports 48 servers per rack with room for growth
- **Simple management:** One subnet = one rack = one leaf switch
- **VLAN alignment:** Each rack gets dedicated VLAN (100-107)
- **Future expansion:** Can subnet further if needed

---

## Hardware Selection

### Spine Switches: Cisco Nexus 9364C

**Specifications:**
- 64x 100GbE QSFP28 ports
- 12.8 Tbps switching capacity
- Sub-microsecond latency

**Reasoning:**
- **Port density:** Can support 16+ leaf switches (we use 8)
- **Performance:** Line-rate forwarding at 100G
- **Feature set:** Full BGP support, VxLAN capable

### Leaf Switches: Cisco Nexus 93180YC-FX

**Specifications:**
- 48x 25GbE SFP28 (server-facing)
- 6x 100GbE QSFP28 (uplinks)
- 3.6 Tbps switching capacity

**Reasoning:**
- **Server connectivity:** 25GbE provides adequate bandwidth for modern servers
- **Oversubscription:** 4:1 (1.2 Tbps down, 400 Gbps up with 4 uplinks)
- **Redundancy:** 4 uplinks provide full redundancy to spine layer

---

## Redundancy & High Availability

### Full Mesh Topology

**Decision:** Connect every leaf to every spine (32 total links).

**Reasoning:**
- **No single point of failure:** Loss of any spine or leaf doesn't partition network
- **Maximum bandwidth:** All links actively forward traffic (no STP blocking)
- **Fast convergence:** BGP reconverges in <1 second on link failure

### ECMP Configuration

**Decision:** Configure `maximum-paths 4` on leaves, `maximum-paths 8` on spines.

**Reasoning:**
- Leaves have 4 uplinks → can use all 4 paths simultaneously
- Spines have 8 downlinks → can load balance across all 8 leaves
- Traffic automatically redistributes when paths fail

---

## Scalability Considerations

### Current Capacity
- **Servers:** 384 ports (48 per leaf × 8 leaves)
- **Bandwidth:** 9.6 Gbps aggregate server-facing (48 × 25G × 8)
- **Spine capacity:** 3.2 Tbps (800G per spine × 4)

### Growth Path
1. **Phase 1:** Fill existing 8 leaves (384 servers)
2. **Phase 2:** Add 4 more leaves to existing spines (768 servers total)
3. **Phase 3:** Add spine capacity if needed (12-16 leaves max per 4 spines)

### Design Headroom
- Spines support up to 16 leaves with 4 uplinks each
- Current design uses 8 leaves = 50% spine capacity utilized
- Room to double server count without adding spines

---

## Security Considerations

### Network Segmentation
- Each rack isolated to separate VLAN and subnet
- Inter-rack traffic flows through leaf switches (can apply ACLs)
- Management network separated (10.255.0.0/24)

### BGP Security
- Use MD5 authentication on BGP sessions (not shown in configs)
- Prefix filtering to prevent route leaks
- Maximum prefix limits per neighbor

---

## Operational Simplicity

### Configuration Templates
- All spines use identical config structure (only IPs/AS change)
- All leaves use identical config structure (only IPs/AS change)
- Enables automation via Ansible/Python

### Naming Conventions
- Devices: SPINE-[1-4], LEAF-[1-8]
- Interfaces: Descriptive names showing remote end
- IP scheme: Predictable pattern (10.1.[spine].[leaf×2])

This design prioritizes **reliability, scalability, and operational simplicity** over upfront cost savings.
