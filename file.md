# Building a Secure 2‑Tier Web Application on Azure (IaaS)

**Author:** Dylan  
**Role:** Cloud Security Specialist  
**Video Walkthrough:** [Watch on Loom](https://www.loom.com/share/6613a59b3aea4c168d60100e2f4f6383)

---

## Overview

This project demonstrates how to design and deploy a **secure two‑tier architecture** in **Microsoft Azure** using Infrastructure‑as‑a‑Service (IaaS).

The deployment consists of a **public web tier** and a **private database tier**, connected through a single **Virtual Network (VNet)**.  
It follows key cloud security best practices, including **network segmentation**, **least‑privilege access**, and **controlled inbound connectivity**.

### Architecture Summary
- **Web Tier (Public Subnet):** Web Server `vm-web-01`
- **Database Tier (Private Subnet):** Database Server `vm-db-01`
- **Virtual Network:** Connects both tiers
- **Network Security Group (NSG):** Restricts access so only the web tier can reach the database

---

## Objectives

- Deploy a secure 2‑tier Azure environment  
- Configure subnets for public and private workloads  
- Use SSH key authentication for secure access  
- Validate internal connectivity  
- Apply NSG rules to restrict database access  

---

## Technologies Used

- **Azure Virtual Network (VNet)**
- **Azure Subnets**
- **Azure Virtual Machines (Ubuntu 20.04 LTS)**
- **SSH Key Authentication**
- **Network Security Groups (NSGs)**
- **Azure Portal**

---

## Phase 1 — Network Architecture

**Resource Group:** `rg-lab02-dylan`  
**VNet Address Space:** `10.0.0.0/16`

**Subnets:**
- Web Subnet → `10.0.1.0/24`
- Database Subnet → `10.0.2.0/24`

This layout allows for easy scaling of future application tiers or service integrations.

---

## Phase 2 — Deploy the Web Server (vm‑web‑01)

| Setting | Value |
|----------|-------|
| Image | Ubuntu Server 20.04 LTS |
| Size | Standard_B1s |
| Authentication | SSH key (`key-lab02.pem`) |
| Subnet | `snet-web` |
| Public IP | 20.121.42.64 |
| Inbound Ports | SSH (22), HTTP (80) |

**SSH into the web server:**
```bash
ssh -i key-lab02.pem azureuser@20.121.42.64
