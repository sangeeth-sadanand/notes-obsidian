---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Azure/Azure|Azure]]"
tags:
  - DataBricks/Azure
index: 2
type: topic
---
# Compute services

```ad-summary
collapse: true 
title: Summary

## 1. Virtual Machines (VMs)

Azure VMs solve the traditional physical server problems of **underutilization** (wasted resources) and **application collision** (software conflicts).

- **How it works**
	- Azure uses a hypervisor to split a single physical server into multiple isolated VMs. 
	- Each VM has its own independent virtual hardware and OS. 
	- If one crashes, the others remain unaffected.
    
- **Management** 
	- Because VMs are Infrastructure as a Service (IaaS), you rent the hardware but retain full control over the OS. 
	- This means you are completely responsible for patching, securing, and maintaining the software.
## 2. Virtual Machine Scale Sets (VMSS)

VMSS simplifies the deployment and management of large groups of identical, load-balanced VMs.

- **Central Management:** 
	- You configure a single template, and VMSS automatically creates exact copies.
- **Auto-scaling:** 
	- VMSS dynamically adds VMs (scales out) during high traffic and removes them (scales in) during low traffic based on metrics like CPU usage. 
	- This optimizes performance while minimizing costs.

## 3. Availability Sets (Intra-Datacenter Protection)

Availability Sets protect your application from localized hardware failures or maintenance downtime **within a single datacenter**. They do this by distributing your VMs across two boundaries:

- **Fault Domains** 
	- Physical separation. 
	- VMs are spread across different server racks (up to 3) with separate power and network switches to survive hardware failures.
- **Update Domains:** 
	- Logical separation. 
	- VMs are grouped into staggered maintenance schedules (up to 20) so that Azure never reboots all your servers for patching at the same time.

## Availability Sets vs. Availability Zones

While both ensure high availability, they protect against different scopes of failure.

| Feature               | Availability Sets                                    | Availability Zones                                         |
| --------------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| **Protection Scope**  | **Intra-datacenter:** Localized within one building. | **Inter-datacenter:** Across different physical buildings. |
| **Protects Against**  | Hardware rack failures and host patching reboots.    | Regional disasters (fires, floods, grid failures).         |
| **Physical Distance** | Feet apart (different racks).                        | Miles apart (different datacenters).                       |
| **Uptime SLA**        | 99.95% (with Managed Disks).                         | 99.99%.                                                    |
| **Cost**              | Free.                                                | Free, but inter-zone data transfer incurs fees.            |


```

---
## 1. Virtual Machines (VMs)

Before virtualization, companies ran applications on "bare-metal" physical servers. This caused two major issues:

- **Underutilization:** A server might only use 10% of its CPU, wasting the rest of the expensive hardware.
- **Application Collision:** Running multiple different applications on one physical server often led to software conflicts, memory leaks, or crashes.
**How VMs solve this:**
Azure uses a piece of software called a **Hypervisor** to carve a single massive physical server into multiple isolated Virtual Machines.
- Each VM acts as its own independent computer with its own virtual CPU, memory, and operating system.
- If one VM crashes, it does not affect the other VMs on the same physical server (solving collision).
- Azure can pack multiple VMs onto one host, maximizing hardware efficiency (solving underutilization).
Because VMs are **Infrastructure as a Service (IaaS)**, you are renting the virtual hardware. You have total control over the operating system, but you are also responsible for patching it, securing it, and installing the software you need.

## 2. Virtual Machine Scale Sets (VMSS)

Managing one VM is easy. Managing 100 identical VMs behind a web application is a logistical nightmare.

**How VMSS solves this:**

A Scale Set allows you to deploy and manage a group of identical, load-balanced VMs as a single resource.

- **Central Management:** You configure a single "template" (OS image, network settings, VM size), and VMSS automatically stamps out exact copies.
- **Auto-scaling:** This is the killer feature. You can set rules based on metrics. For example: _"If average CPU usage hits 80%, add 2 more VMs (Scale Out). If it drops below 20%, remove 2 VMs (Scale In)."_ This ensures you always have enough performance during high traffic, but you aren't paying for idle servers during off-hours.

## 3. Availability Sets (Intra-Datacenter Protection)

While Availability _Zones_ (which we discussed earlier) protect you if an entire datacenter catches fire, Availability _Sets_ protect your VMs from localized failures happening _inside_ a single datacenter.

When you group two or more VMs into an Availability Set, Azure automatically distributes them across two distinct boundaries:

- **Fault Domains (Hardware Protection):** Think of a Fault Domain as a physical server rack. It shares a common power source and a common network switch. If a power supply blows up, that whole rack goes offline. Azure guarantees your VMs are spread across up to 3 different Fault Domains so a single hardware failure won't take down your whole application.
- **Update Domains (Patching Protection):** Microsoft regularly reboots physical host servers to patch security vulnerabilities. An Update Domain is a logical group of hardware that can undergo maintenance at the same time. Azure spreads your VMs across up to 20 Update Domains, sequencing the reboots so your VMs are never all offline for patching at the same time.

## Availability Zone Vs Availability Sets

| Feature                           | Availability Set                                                   | Availability Zone                                                                  |
| --------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| **Scope of Protection**           | Intra-datacenter (Within the same building)                        | Inter-datacenter (Across different buildings)                                      |
| **Protects Against**              | Hardware failures (rack power/network switch) and host patching.   | Datacenter-level disasters (fires, floods, regional power grid failures).          |
| **Physical Distance**             | VMs are a few feet away from each other on different server racks. | VMs are miles away from each other in distinct physical datacenters.               |
| **Service Level Agreement (SLA)** | **99.95%** uptime guarantee (when using Managed Disks).            | **99.99%** uptime guarantee.                                                       |
| **Cost**                          | Free (You only pay for the VMs themselves).                        | Free for the zone itself, but inter-zone network traffic incurs bandwidth charges. |
