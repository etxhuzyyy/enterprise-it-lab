# Acme Solutions Ltd. - Network Plan

## Network Overview

Acme Solutions Ltd. uses a segmented enterprise network designed to separate management systems, employee workstations, and servers.

The network uses VLANs and dedicated IPv4 subnets to provide logical separation between different categories of systems. Inter-VLAN communication will be controlled through routing and firewall policies.

The design is intentionally sized for the simulated 150-employee environment while remaining manageable for a sole IT administrator.

**Domain:** `acme.local`

**Primary DNS Server:** `10.10.0.10` (DC01)

---

## VLAN and Subnet Plan

| VLAN | Network Name | Subnet | Gateway | Purpose |
|---:|---|---|---|---|
| 10 | Management | `10.10.0.0/24` | `10.10.0.1` | Domain controller and IT management systems |
| 20 | Workstations | `10.10.1.0/24` | `10.10.1.1` | Employee workstations |
| 30 | Servers | `10.10.2.0/24` | `10.10.2.1` | File and management servers |

---

## IP Address Assignments

### Management - VLAN 10

| Device | IP Address | Assignment | Purpose |
|---|---|---|---|
| Gateway | `10.10.0.1` | Static | VLAN gateway |
| DC01 | `10.10.0.10` | Static | Active Directory and DNS |
| IT-ADMIN | `10.10.0.20` | Static/Reserved | IT administration |

### Workstations - VLAN 20

| Device | IP Address | Assignment | Purpose |
|---|---|---|---|
| CLIENT01 | `10.10.1.x` | DHCP | Employee workstation |
| CLIENT02 | `10.10.1.x` | DHCP | Employee workstation |

**DHCP Range:** `10.10.1.100 - 10.10.1.200`

Workstations will receive their IP address, subnet mask, default gateway, and DNS configuration through DHCP.

### Servers - VLAN 30

| Device | IP Address | Assignment | Purpose |
|---|---|---|---|
| Gateway | `10.10.2.1` | Static | VLAN gateway |
| SERVER02 | `10.10.2.20` | Static | File Server |
| SERVER03 | `10.10.2.30` | Static | Management Server |

---

## Core Services

| Service | Host | IP Address |
|---|---|---|
| Active Directory Domain Services | DC01 | `10.10.0.10` |
| DNS | DC01 | `10.10.0.10` |
| DHCP | TBD | TBD |
| File Services | SERVER02 | `10.10.2.20` |
| Management Services | SERVER03 | `10.10.2.30` |

---

## DNS and Domain

**Active Directory Domain:** `acme.local`

**Primary DNS Server:** `10.10.0.10`

DC01 will provide DNS services required for Active Directory name resolution and domain operations.

---

## Network Segmentation

The environment uses three logical network segments:

- **VLAN 10 - Management:** Reserved for domain infrastructure and IT administration.
- **VLAN 20 - Workstations:** Used by employee endpoints.
- **VLAN 30 - Servers:** Used by enterprise servers and shared services.

Inter-VLAN communication will be routed and controlled using appropriate firewall and access-control policies.

---

## Design Goals

The network design aims to:

1. Separate users, servers, and management systems.
2. Provide centralized DNS and Active Directory services.
3. Use DHCP for employee workstations.
4. Assign static addresses to infrastructure systems.
5. Support centralized administration by a sole IT administrator.
6. Provide a foundation for security controls and access restrictions.
7. Allow future expansion without redesigning the entire addressing scheme.

All systems and network configurations in this project are simulated for educational and portfolio purposes.