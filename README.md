Building a Secure 2‑Tier Web Application (Azure IaaS)
Course Author: Dylan Bryson 
📌 Overview
This project demonstrates how to build a secure 2‑tier IaaS architecture in Microsoft Azure.
The design includes:

A public subnet hosting a Web Server (vm-web-01)

A private subnet hosting a Database Server (vm-db-01)

A Virtual Network (VNet) connecting both tiers

A Network Security Group (NSG) restricting DB access so only the Web subnet can reach it

This architecture models real‑world cloud deployments where front‑end and back‑end systems must be isolated and protected.

flowchart TD

    Internet(["🌐 Internet"])
    PublicIP["Public IP: 20.121.42.64"]

    subgraph WebSubnet["snet-web (10.0.1.0/24)"]
        WebVM["vm-web-01"]
    end

    subgraph DBSubnet["snet-db (10.0.2.0/24)"]
        DBVM["vm-db-01 (10.0.2.5)"]
    end

    Internet --> PublicIP --> WebSubnet
    WebSubnet -->|"Private VNet Communication"| DBSubnet

🎯 Objectives
Deploy a secure 2‑tier Azure environment

Configure subnets for public and private workloads

Use SSH to access a web server

Validate internal connectivity

Apply NSG rules to restrict access to the database tier

🛠️ Technologies Used
Azure Virtual Network (VNet)

Azure Subnets

Azure Virtual Machines (Ubuntu 20.04 LTS)

SSH Key Authentication

Network Security Groups (NSGs)

Azure Portal

📐 Phase 1 — Create the Network Foundation
Resources Created
Resource Group: rg-lab02-dylan

VNet: vnet-lab02

Address Space: 10.0.0.0/16

Subnets:

snet-web → 10.0.1.0/24 (Public)

snet-db → 10.0.2.0/24 (Private)

This establishes the network boundaries for the web and database tiers.

🌐 Phase 2 — Deploy the Web Server (vm‑web‑01)
Configuration
Image: Ubuntu Server 20.04 LTS

Size: Standard_B1s

Authentication: SSH Key (key-lab02.pem)

Subnet: snet-web

Public IP: 20.121.42.64

Inbound Ports:

SSH (22)

HTTP (80)

SSH Command
bash
ssh -i key-lab02.pem azureuser@20.121.42.64
This VM acts as the entry point into the environment.

🗄️ Phase 3 — Deploy the Database Server (vm‑db‑01)
Configuration
Image: Ubuntu Server 20.04 LTS

Size: Standard_B1s

Authentication: Same SSH key

Subnet: snet-db

Private IP: 10.0.2.5

Public IP: None (Private-only server)

This ensures the DB server is isolated from the internet.

🔗 Phase 4 — Validate Internal Connectivity
SSH into the web server:

bash
ssh -i key-lab02.pem azureuser@20.121.42.64
Ping the DB server’s private IP:

bash
ping 10.0.2.5
Successful replies confirm the VNet and subnets are configured correctly.

🛡️ Phase 5 — Configure the Firewall (NSG)
Open the NSG attached to vm‑db‑01 and add this inbound rule:

Setting	Value
Source	IP Addresses
Source Range	10.0.1.0/24
Destination	Any
Destination Port	*
Action	Allow
Priority	100
Name	Allow-Web-Subnet

Purpose
This rule ensures only the Web subnet can communicate with the DB server — a core security requirement in 2‑tier architectures.

🧹 Cleanup
To remove all resources:

Go to Resource Groups

Delete: rg-lab02-dylan

This deletes the entire lab environment.

📘 Summary
This lab demonstrates how to:

Build a secure 2‑tier architecture in Azure

Use subnets to isolate workloads

Use NSGs to enforce least‑privilege access

Validate internal connectivity using SSH and ping

This is a foundational cloud architecture pattern used in real production environments.
