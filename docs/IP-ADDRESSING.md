# IP Addressing Scheme

## Overview
This document details the complete IP addressing plan for the data center spine-leaf network.

## Address Space Allocation

| Network Type | Subnet | Purpose |
|-------------|---------|----------|
| Spine Loopbacks | 10.0.0.0/28 | BGP router IDs for spine switches |
| Leaf Loopbacks | 10.0.0.16/28 | BGP router IDs for leaf switches |
| P2P Links | 10.1.0.0/16 | Point-to-point spine-leaf interconnects |
| Server Subnets | 10.10.0.0/16 | Production server networks |
| Management | 10.255.0.0/24 | Out-of-band management network |

## Spine Switch Assignments

| Device | Loopback0 | BGP AS | Management IP |
|--------|-----------|--------|---------------|
| SPINE-1 | 10.0.0.1/32 | 65001 | 10.255.0.1/24 |
| SPINE-2 | 10.0.0.2/32 | 65002 | 10.255.0.2/24 |
| SPINE-3 | 10.0.0.3/32 | 65003 | 10.255.0.3/24 |
| SPINE-4 | 10.0.0.4/32 | 65004 | 10.255.0.4/24 |

## Leaf Switch Assignments

| Device | Loopback0 | BGP AS | Server Subnet | Management IP |
|--------|-----------|--------|---------------|---------------|
| LEAF-1 | 10.0.0.11/32 | 65011 | 10.10.1.0/24 | 10.255.0.11/24 |
| LEAF-2 | 10.0.0.12/32 | 65012 | 10.10.2.0/24 | 10.255.0.12/24 |
| LEAF-3 | 10.0.0.13/32 | 65013 | 10.10.3.0/24 | 10.255.0.13/24 |
| LEAF-4 | 10.0.0.14/32 | 65014 | 10.10.4.0/24 | 10.255.0.14/24 |
| LEAF-5 | 10.0.0.15/32 | 65015 | 10.10.5.0/24 | 10.255.0.15/24 |
| LEAF-6 | 10.0.0.16/32 | 65016 | 10.10.6.0/24 | 10.255.0.16/24 |
| LEAF-7 | 10.0.0.17/32 | 65017 | 10.10.7.0/24 | 10.255.0.17/24 |
| LEAF-8 | 10.0.0.18/32 | 65018 | 10.10.8.0/24 | 10.255.0.18/24 |

## Point-to-Point Link Addressing

Using /31 subnets (RFC 3021) for efficient IP usage on point-to-point links.

### Naming Convention
Format: `10.1.[SPINE].[LEAF*2]/31`
- SPINE = Spine switch number (1-4)
- LEAF = (Leaf number - 1) × 2

### Examples

| Connection | Subnet | Spine IP | Leaf IP |
|-----------|---------|----------|---------|
| SPINE-1 ↔ LEAF-1 | 10.1.1.0/31 | 10.1.1.0 | 10.1.1.1 |
| SPINE-1 ↔ LEAF-2 | 10.1.1.2/31 | 10.1.1.2 | 10.1.1.3 |
| SPINE-2 ↔ LEAF-1 | 10.1.2.0/31 | 10.1.2.0 | 10.1.2.1 |
| SPINE-2 ↔ LEAF-2 | 10.1.2.2/31 | 10.1.2.2 | 10.1.2.3 |

*Pattern continues for all 32 point-to-point links (4 spines × 8 leaves)*

## BGP AS Number Allocation

### Design Decision
Using unique AS numbers per switch enables:
- Simple eBGP configuration
- Automatic ECMP (equal-cost multipath)
- Easy troubleshooting via AS-path
- Horizontal scalability

### AS Number Ranges
- **Spines:** AS 65001-65004
- **Leaves:** AS 65011-65018
- **Range:** Private AS numbers (64512-65534 per RFC 6996)

## Server Networks

Each leaf switch serves one /24 subnet:
- **Capacity:** 254 usable IPs per rack
- **Gateway:** .1 address (on leaf switch)
- **DHCP Range:** .10-.250 (suggested)
- **Static Range:** .251-.254 (for infrastructure)

### Example: LEAF-1 Server Network
- **Subnet:** 10.10.1.0/24
- **Gateway:** 10.10.1.1 (LEAF-1 VLAN 100)
- **Servers:** 10.10.1.10 - 10.10.1.250
