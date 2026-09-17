---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Azure/Azure|Azure]]"
tags:
  - DataBricks/Azure
index: 3
type: topic
---
# Resource group

```ad-summary
collapse: true 
title: Summary
## Resource Group
- A Resource Group is a logical container that holds related resources for an Azure solution. 
- To manage it effectively, keep these core rules in mind:
	- **Single Ownership**
	- **No Nesting**
	- **High Mobility**
	- **Metadata Location**
	- **Geographic Flexibility**
	
> [!Tip]
> Because resources can be in different locations than their group, the resource group's location is primarily a compliance and residency boundary for your deployment logs, not the actual application traffic.

```
---
# Resource Group
A **Resource Group** is a logical container that holds related resources for an Azure solution. To manage it effectively, keep these core rules in mind:

- **Single Ownership:** A resource can belong to **only one** resource group at a time.
- **No Nesting:** Resource groups cannot be nested inside other resource groups.
- **High Mobility:** You can easily move (shuffle) most resources between different resource groups when your application architecture changes.
- **Metadata Location:** Every resource group must be assigned a location (region). This is strictly where the deployment **metadata** and history are stored.
- **Geographic Flexibility:** Resources inside a resource group do **not** have to match the group's location. For example, a resource group based in _East US_ can contain a Virtual Machine running in _West Europe_.

> [!tip]
> Because resources can be in different locations than their group, the resource group's location is primarily a compliance and residency boundary for your deployment logs, not the actual application traffic.
