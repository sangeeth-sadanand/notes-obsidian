---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Azure/Azure|Azure]]"
tags:
  - DataBricks/Azure
index: 4
type: topic
---
# Global Infrastructure

```ad-summary
collapse: true 
title: Summary

![[005_assets/Big_Data/Global_Infrastructure.drawio.svg]]
## Data Center
- The foundational physical facility housing servers, custom cooling, independent power generation, and networking.

## Availability Zone (AZ)
- Unique, isolated physical locations within a region, made up of one or more data centers with independent power, cooling, and networking. 
- Regions supporting AZs must have at least three zones. Data is replicated **synchronously** across zones for real-time high availability (99.99% SLA).

## Region 
- A latency-defined perimeter (under 2ms) containing multiple data centers.
- Regions are categorized as _Recommended_ (full features/AZs) or _Alternate_ (specific workloads). 
- **Selection criteria** include user proximity, service availability, and cost-effectiveness.

## Region Pair
- Two regions within the same geography situated at least 300 miles apart. 
- Designed for disaster recovery using **asynchronous** replication. 
- Azure ensures platform updates are applied sequentially across pairs to prevent simultaneous outages.

## Geography
- Geopolitical boundaries (e.g., US, Europe, India) designed to enforce data residency, sovereignty, and compliance. 
- This includes isolated Sovereign Clouds like Azure Government and Azure China.
  
## Availability Zone Service Categories

Azure services fall into three deployment categories:

| Service Type              | How It Works                                                   | Best For                                                                                | Examples                                               |
| ------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Zonal**                 | Pinned to a specific, single Availability Zone.                | Minimizing latency by co-locating dependent resources in the exact same physical space. | Virtual Machines (VMs), Managed Disks, Public IPs      |
| **Zone-Redundant**        | Automatically replicated across multiple zones by Azure.       | Zero data-loss failover and out-of-the-box high availability without manual management. | Azure SQL, Zone-Redundant Storage (ZRS), VM Scale Sets |
| **Non-Regional (Global)** | Distributed globally; not tied to any specific region or zone. | Global traffic routing, edge security, and universal identity management.               | Entra ID, Traffic Manager, Azure Front Door, DNS       |
```
---
# Global Infrastructure
![[005_assets/Big_Data/Global_Infrastructure.drawio.svg]]

## 1. Data Center

- **The Physical Unit:** A single facility housing thousands of blades/servers arranged in racks.
	
- **Hyper-Scale Footprint:** Azure data centers are organized into "clusters" inside the building. Microsoft custom-designs these facilities down to the cooling systems (often using evaporative cooling or liquid cooling for high-density AI clusters).
	
- **Power & Networking:** Features autonomous power generation (generators and massive UPS systems) and is connected to Microsoft’s global fiber-optic network.

## 2. Region

- **Definition:** A geographical boundary containing a set of data centers deployed within a latency-defined perimeter.
	
- **Latency Guarantee:** Data centers within a region are connected via a dedicated, regional low-latency network. This latency is typically **under 2 milliseconds**.
	
- **Global Footprint:** Azure has more global regions than any other cloud provider (over 60+ regions worldwide).
	
- **Service Availability Tiers:** Not all regions are identical. Azure categorizes regions into:
	
	- **Recommended Regions:** Designed to support Availability Zones and the full suite of Azure services.
		
	- **Alternate Regions:** Optimized for specific workloads, disaster recovery (as part of a pair), or local data residency; they have a more limited service catalog.

## 3. Availability Zone (AZ)

- **Isolation Boundary:** A zone is a unique physical location _within_ a region. Each zone is made up of one or more data centers equipped with independent power, cooling, and networking.
	
- **High Availability ($99.99\%$ SLA):** By deploying VMs or services across multiple zones (Zone-Redundant or Zonal deployments), you protect your application from a total data center outage.
	
- **Synchronous Replication:** Data replicated across Availability Zones within the same region is done **synchronously**. Because the physical distance between zones is minimal, write operations are committed across zones in real-time without hurting application performance.
	
- **Minimum Requirement:** Any Azure region that supports Availability Zones must have a **minimum of three separate zones**.

## 4. Region Pair

- **The 300-Mile Rule:** Each Azure region is statically paired with another region within the same geography, typically located at least **300 miles (480 km)** apart to isolate them from regional disasters (e.g., hurricanes, earthquakes, massive power grid failures).
	
- **Asynchronous Replication:** Unlike Availability Zones, data replication between region pairs (e.g., GRS - Geo-Redundant Storage) happens **asynchronously** to avoid performance bottlenecks over long distances.
	
- **Platform Update Sequencing:** During planned Azure maintenance, updates are rolled out sequentially. Azure will never update both regions in a pair at the same time, ensuring one environment is always active.
	
- **Disaster Recovery Prioritization:** If a massive global outage occurs, Azure prioritizes the recovery of at least one region in every pair to restore critical services quickly.

## 5. Geography

- **Geopolitical Boundaries:** Typically aligned with country borders or legal jurisdictions (e.g., United States, India, Europe, Canada).
	
- **Data Residency & Sovereignty:** Many industries (Finance, Healthcare, Government) legally prohibit data from leaving national borders. Geographies allow customers with strict data-residency requirements to keep their data and applications close to home.
	
- **Sovereign/Specialized Clouds:** Beyond standard geographies, Azure operates completely isolated physical networks for specific legal domains:
	
	- **Azure Government:** Physically isolated data centers dedicated strictly to US Federal, State, and Local government agencies and their contractors.
		
	- **Azure China:** Operated independently by **21Vianet** to comply with strict Chinese telecommunications and internet laws.

## Azure Region Selection Criteria
- Region should be near to the end-user to maintain low latency
- Not all region provide all service so region should be selected which provide the required service
- Prices may vary across the region.
- Select the region which is cost-effective and maximum service

## Categories of Availability Zone
### 1. Zonal Services

- **What they are:** You explicitly choose a specific, single Availability Zone to deploy the resource into. This pins the resource to that exact zone's data center infrastructure.
- **Examples:** Virtual Machines (VMs), Managed Disks, Public IP addresses.
- **Use Case:** Ideal when you want to minimize latency by co-locating different resources (like a VM and its disk) in the exact same physical zone.

### 2. Zone-Redundant Services
- **What they are:** Azure automatically replicates the service and its data across multiple Availability Zones within the region. You don't need to manage the distribution yourself.
- **Examples:** Azure SQL Database, Azure Storage accounts (specifically ZRS - Zone-Redundant Storage), Virtual Machine Scale Sets.
- **Use Case:** High availability out of the box. If one zone experiences a failure, the service automatically fails over to another zone with zero data loss.
    
### 3. Non-Regional (Global) Services
- **What they are:** These services are not tied to a specific Azure region or Availability Zone at all. They are distributed globally by Microsoft to ensure continuous availability.
- **Examples:** Azure Active Directory (Microsoft Entra ID), Azure Traffic Manager, Azure Front Door, Azure DNS.
- **Use Case:** Global traffic routing, identity management, and edge-security.
