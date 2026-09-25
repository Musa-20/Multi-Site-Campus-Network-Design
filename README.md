# Multi-Site-Campus-Network-Design
Designed and implemented a multi-site university network in Cisco Packet Tracer with VLAN segmentation, OSPF routing, DHCP/DNS services, ACLs, SSH, NAT/PAT, and a remote branch. 

The network consists of:
- 3 campus buildings:
  - Engineering
  - Business
  - Library
- 7 VLANs for user and management segmentation
- 120+ campus endpoints
- Centralized DHCP, DNS, Web, and File servers
- ISP connection and simulated Internet
- Remote branch with 3 PCs
- Core Layer 3 switching and routing

## 1. What I Worked On

### VLAN Segmentation

Created separate VLANs for different departments and user groups:

| VLAN | Purpose | Network |
|---|---|---|
| 10 | Engineering Faculty/Staff | 172.16.10.0/24 |
| 20 | Engineering Students | 172.16.20.0/24 |
| 30 | Business Faculty/Staff | 172.16.30.0/24 |
| 40 | Business Students | 172.16.40.0/24 |
| 50 | Library Staff/Admin | 172.16.50.0/24 |
| 60 | Library Public | 172.16.60.0/24 |
| 99 | Network Management | 172.16.99.0/24 |

Inter-VLAN routing was implemented using the Layer 3 core switch.

### Routing

Implemented:

- OSPF Area 0
- Default routing toward the ISP
- Routing between the campus and remote branch
- OSPF default route advertisement

### Network Services

Configured centralized:

- DHCP
- DHCP relay
- DNS
- Web server
- File server

The DHCP server provides addressing information to the different campus VLANs through DHCP relay configuration.

### Network Security

Implemented:

- SSHv2 for device management
- Management VLAN
- Standard and Extended ACLs
- VTY access restrictions
- VLAN segmentation
- Restricted administrative access

### NAT and Remote Branch

Configured:

- NAT/PAT for campus users
- Static NAT for the campus web server
- NAT for the remote branch
- ISP connectivity
- Branch-to-campus routing

The remote branch uses the `192.168.70.0/24` network.

## 2. Troubleshooting Issues

A major part of the project involved identifying and fixing configuration and connectivity problems.

## Issue 1 — SSH Access Failure

### Problem

The VTY access-class ACL contained an incorrect network and wildcard mask:

```text
192.16.99.0 0.0.0.155
```

The actual management network was:

```text
172.16.99.0/24
```

### Fix

Corrected the ACL to ```172.16.99.0/24```

SSH access from the management VLAN then worked as expected.

## Issue 2 — Remote Branch Routing Failure

### Problem

The remote branch could reach its ISP connection but could not properly reach the campus network.

Using `show ip route`, `ping`, and `tracert`, the missing route to the branch WAN network was identified.

### Fix

On the branch router, PAT was configured which the core router had... 

Added the following route to the core router:

```text
ip route 203.0.114.0 255.255.255.252 203.0.113.1
```

## Issue 3 — Double NAT on the Remote Branch

### Problem

The branch network `192.168.70.0/24` was included in the core router's NAT configuration even though the branch router was already performing NAT.

This resulted in the branch traffic being NATed twice and lead to remote branch pc having a 50% loss in pings to the devices in the campus branch..

### Fix

Removed the branch network from the core router's NAT ACL:

```text
no access-list 1 permit 192.168.70.0 0.0.0.255
```

The branch router remained responsible for NATing its own LAN traffic.

## Issue 4 — Double NAT on the Remote Branch

### Problem

The branch network `192.168.70.0/24` was included in the core router's NAT configuration even though the branch router was already performing NAT.

This resulted in the branch traffic being NATed twice and lead to remote branch pc having a 50% loss in pings to the devices in the campus branch..

### Fix

Removed the branch network from the core router's NAT ACL:

```text
no access-list 1 permit 192.168.70.0 0.0.0.255
```

The branch router remained responsible for NATing its own LAN traffic.

## Issue 5 — ACLs Not Applied to the Intended SVI

### Problem

While configuring VLAN security, the ACLs were created successfully but were not being applied to the intended Switch Virtual Interfaces (SVIs). As a result, the ACL rules were not affecting traffic for the corresponding VLANs.

This seemed to be an issue with the software running on MacOS. Others on the internet have reported they saw the same issue when running on MacOS but the issue did not come up when running on Windows

