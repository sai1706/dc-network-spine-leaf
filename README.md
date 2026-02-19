# Data Center Spine-Leaf Network Design

Production-grade data center network using spine-leaf architecture with eBGP routing.

## Project Overview
- **Capacity:** 600 servers across 8 racks
- **Architecture:** 4 Spine + 8 Leaf switches
- **Routing:** eBGP with unique AS per switch
- **Redundancy:** Full mesh, every leaf connects to every spine
- **ECMP:** 4-way load balancing

## Design Highlights
- 4 Spine Switches: Cisco Nexus 9364C (100GbE)
- 8 Leaf/TOR Switches: Cisco Nexus 93180YC (48x 25GbE + 6x 100GbE)
- 32 point-to-point links (4 spines × 8 leaves)
- BGP AS Range: 65001-65004 (spines), 65011-65018 (leaves)

## Repository Contents
- **[Network Diagram](dc-topology.drawio.png)** - Spine-leaf topology visualization
- **[Configurations](/configs)** - Device configs for spines and leaves
  - [SPINE-1](/configs/spine-1.cfg) | [SPINE-2](/configs/spine-2.cfg)
  - [LEAF-1](/configs/leaf-1.cfg) | [LEAF-2](/configs/leaf-2.cfg)
- **[Documentation](/docs)**
  - [IP Addressing Scheme](/docs/IP-ADDRESSING.md)
  - [Design Decisions](/docs/DESIGN-DECISIONS.md)

## Key Features
- Full mesh connectivity (32 P2P links)
- eBGP routing with unique AS per switch
- ECMP load balancing across all paths
- /31 subnets for efficient IP usage
- Scalable to 768 servers (16 leaves)
---
**Author:** sai1706
**Project Date:** February 2025
