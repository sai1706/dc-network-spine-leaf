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
- `/diagrams` - Network topology and traffic flow diagrams
- `/configs` - Complete device configurations for all switches
- `/docs` - IP addressing tables and design decisions

## Status
🚧 **In Progress** - Building topology and configurations

---
**Author:** sai1706
**Project Date:** February 2025
