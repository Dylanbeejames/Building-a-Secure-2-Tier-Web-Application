# Building a Secure 2-Tier Web Application on Azure (IaaS)

**Author:** Dylan   
**Video Walkthrough:** [▶ Watch on Loom](https://www.loom.com/share/6613a59b3aea4c168d60100e2f4f6383)

---

## Table of Contents

- [Overview](#overview)
- [Architecture Summary](#architecture-summary)
- [Objectives](#objectives)
- [Technologies Used](#technologies-used)
- [Phase 1 — Network Architecture](#phase-1--network-architecture)
- [Phase 2 — Deploy the Web Server](#phase-2--deploy-the-web-server-vm-web-01)
- [Phase 3 — Deploy the Database Server](#phase-3--deploy-the-database-server-vm-db-01)
- [Phase 4 — Validate Internal Connectivity](#phase-4--validate-internal-connectivity)
- [Phase 5 — NSG Configuration](#phase-5--network-security-group-nsg-configuration)
- [Phase 6 — Cleanup](#phase-6--cleanup)
- [Architecture Diagram](#architecture-diagram)
- [Summary](#summary)
- [Next Steps](#next-steps)

---

## Overview

This project demonstrates how to design and deploy a **secure two-tier architecture** in **Microsoft Azure** using Infrastructure-as-a-Service (IaaS).

The deployment consists of a **public web tier** and a **private database tier**, connected through a single **Virtual Network (VNet)**. It follows key cloud security best practices, including **network segmentation**, **least-privilege access**, and **controlled inbound connectivity**.

---

## Architecture Summary

| Tier | Component | Subnet |
|------|-----------|--------|
| Web (Public) | `vm-web-01` | `snet-web` — `10.0.1.0/24` |
| Database (Private) | `vm-db-01` | `snet-db` — `10.0.2.0/24` |
| Networking | Azure VNet | `10.0.0.0/16` |
| Security | NSG on `snet-db` | Allow only web tier traffic |

---

## Objectives

- Deploy a secure 2-tier Azure environment
- Configure subnets for public and private workloads
- Use SSH key authentication for secure VM access
- Validate internal connectivity between tiers
- Apply NSG rules to restrict database access

---

## Technologies Used

| Technology | Purpose |
|------------|---------|
| Azure Virtual Network (VNet) | Network backbone for both tiers |
| Azure Subnets | Logical tier segmentation |
| Azure Virtual Machines (Ubuntu 20.04 LTS) | Web and database compute |
| SSH Key Authentication | Secure, passwordless access |
| Network Security Groups (NSGs) | Subnet-level traffic enforcement |
| Azure Portal | Resource provisioning and management |

---

## Phase 1 — Network Architecture

**Resource Group:** `rg-lab02-dylan`  
**VNet Address Space:** `10.0.0.0/16`

| Subnet | Name | CIDR |
|--------|------|------|
| Web (Public) | `snet-web` | `10.0.1.0/24` |
| Database (Private) | `snet-db` | `10.0.2.0/24` |

> **Design Note:** This `/16` address space allows easy horizontal scaling — additional subnets can be introduced for future tiers (e.g., caching, middleware) or service integrations without restructuring.

---

## Phase 2 — Deploy the Web Server (`vm-web-01`)

| Setting | Value |
|---------|-------|
| Image | Ubuntu Server 20.04 LTS |
| Size | Standard_B1s |
| Authentication | SSH key (`key-lab02.pem`) |
| Subnet | `snet-web` |
| Public IP | `20.121.42.64` |
| Inbound Ports | SSH (22), HTTP (80) |

**Connect to the web server:**

```bash
ssh -i key-lab02.pem azureuser@20.121.42.64
```

> This VM serves as the **sole public entry point** into the environment. All external traffic is funneled through this tier.

---

## Phase 3 — Deploy the Database Server (`vm-db-01`)

| Setting | Value |
|---------|-------|
| Image | Ubuntu Server 20.04 LTS |
| Size | Standard_B1s |
| Authentication | SSH key (`key-lab02.pem`) |
| Subnet | `snet-db` |
| Private IP | `10.0.2.5` |
| Public IP | **None** |

> **Security Note:** No public IP is assigned to `vm-db-01`. This VM is fully isolated from the public internet — access is only possible via internal VNet routing from the web tier.

---

## Phase 4 — Validate Internal Connectivity

**Step 1:** SSH into the web server.

```bash
ssh -i key-lab02.pem azureuser@20.121.42.64
```

**Step 2:** From the web server, ping the database VM's private IP.

```bash
ping 10.0.2.5
```

A successful response confirms that **VNet routing between subnets is correctly configured** and the two tiers can communicate internally.

---

## Phase 5 — Network Security Group (NSG) Configuration

An NSG is attached to `snet-db` to enforce the **least-privilege access** principle — only the web subnet is permitted to initiate connections to the database tier.

### Inbound Security Rule

| Setting | Value |
|---------|-------|
| Rule Name | `Allow-Web-Subnet` |
| Source IP Range | `10.0.1.0/24` |
| Destination | Any |
| Destination Port | `*` |
| Protocol | Any |
| Action | **Allow** |
| Priority | `100` |

> All other inbound traffic to `snet-db` is implicitly denied. This ensures the database tier is unreachable from any source outside the web subnet — including the public internet.

---

## Phase 6 — Cleanup

To teardown the lab and avoid incurring unnecessary Azure charges:

1. Open the **Azure Portal**
2. Navigate to **Resource Groups**
3. Locate `rg-lab02-dylan`
4. Click **Delete Resource Group** and confirm

> ⚠️ This permanently removes all resources within the group, including both VMs, the VNet, subnets, and NSG rules.

---

## Architecture Diagram

```
┌─────────────────────────────────────────┐
│            Azure VNet                   │
│           10.0.0.0/16                   │
│                                         │
│  ┌──────────────────────────────────┐   │
│  │  Web Subnet — 10.0.1.0/24        │◄──┼── Public Internet (SSH / HTTP)
│  │  vm-web-01 — 20.121.42.64        │   │
│  └──────────────┬───────────────────┘   │
│                 │ VNet Routing           │
│  ┌──────────────▼───────────────────┐   │
│  │  DB Subnet — 10.0.2.0/24         │   │
│  │  vm-db-01  — 10.0.2.5            │   │
│  │  NSG: Allow-Web-Subnet only      │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## Summary

This lab demonstrates a production-aligned approach to deploying secure cloud workloads on Azure:

- ✅ Deployed a secure two-tier Azure architecture using IaaS
- ✅ Enforced network isolation via subnet segmentation
- ✅ Applied NSG rules following the least-privilege principle
- ✅ Verified internal tier-to-tier communication
- ✅ Followed real-world design patterns for secure cloud environments

This foundational pattern is extensible — it can be evolved with load balancers, WAFs, or managed database services for full production readiness.

---

## Next Steps

| Enhancement | Description |
|-------------|-------------|
| **Azure Load Balancer** | Add high availability to the web tier |
| **Private DNS Zone** | Enable internal name resolution between VMs |
| **Managed Database** | Migrate to Azure Database for MySQL / PostgreSQL |
| **Azure Bastion** | Secure RDP/SSH access without exposing public IPs |
| **Azure Firewall / WAF** | Add application-layer traffic inspection |

---

## Author Notes

This lab is part of an ongoing **cloud security architecture series**, focusing on core design patterns that balance accessibility, scalability, and protection.

All builds follow the **Azure Well-Architected Framework** pillars:

> *Cost Optimization · Operational Excellence · Reliability · Performance Efficiency · Security*

---

*© Dylan — Cloud Security Specialist*
