---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Azure/Azure|Azure]]"
tags:
  - Databricks/cloud
index: 1
type: Group of topic
---
# Fundamentals

```ad-summary
collapse: true 
title: Summary

## Characteristics of Cloud
- Agility
- Scalability
- Elasticity
- Fault tolerance
- Disaster recovery
- High availability

## Advantages of Cloud Computing
- Agility & instance scalable
- Elastic
- Cost effective
- Geo distributed and reliable 
- Disaster recovery

## Hosting and deployment models

- **On-premises** - Compute infrastructure is manage by the company owned servers
	- **Advantages**- Complete control, Predictable ongoing costs
	- **Disadvantages**- High CapEx and OpEx, slow to scale, requires internal human resource to maintain
- **Cloud computing**- Infrastructure are provided by 3rd party cloud providers
	- **Advantages**- Low CapEx and OpEx, Scalable, every thing is managed by cloud providers
	- **Disadvantage**- Less control over hardware, Recurring cost can spiral up
- **Hybrid cloud**- A combination of on-prem and cloud.
	- **Advantages**- Maximum flexibility, allows keeping sensitive data on-perm and use public for catering high demand spikes
	- **Disadvantage**- Complex setup for managing two different environments

## Cloud Service Models
- Infrastructure as a service
- Platform as a service
- Software as a service

## Compute Architecture- Server-Based vs. Serverless

- **Server-Based**-
	- dedicated servers
	- Manual scaling
	- Pay for the resources(RAM/ CPU by hrs / month)
	- full control on the environment and OS
	- Stateful
	- No restriction on processing time
- **Server-less**
	- Resource is allocated for specific time to run the task
	- Auto scaling
	- pay for execution time
	- No control over the underline resource
	- Stateless
	- Strict timeouts
```
---
## Characteristics of Cloud

- **Agility:** Drastically reduces the time required to procure and deploy infrastructure, turning months of hardware setup into minutes of software provisioning.
- **Scalability:** Allows you to scale resources up/down (adjusting power) or in/out (adjusting instances) effortlessly to match shifting demands.
- **Elasticity:** The system's ability to automatically and dynamically allocate or reclaim resources in real time as workloads fluctuate.
- **Fault Tolerance:** Ensures the system continues to operate seamlessly and without interruption, even if individual hardware or software components fail.
- **Disaster Recovery:** Protects operations from natural or human-made disasters by maintaining redundant backup data centers in entirely different geographic locations.
- **High Availability:** Guarantees that applications and services remain up, running, and accessible to users with minimal to zero downtime.

## Advantages of Cloud Computing

Regardless of the specific model, moving to the cloud generally provides these key benefits:

- **Agility & Instant Scalability:** The ability to scale resources up or down almost instantly based on demand.
- **Cost-Effectiveness:** Low upfront costs (CapEx) and the ability to pay only for the resources you actually use.
- **Geo-Distribution:** Deploying applications across multiple global locations to bring data closer to users, significantly reducing **network latency**.
- **Disaster Recovery:** If a server in one geographic location goes down, traffic can be instantly routed to a backup server in another region to ensure high availability.

| Advantage                      | Core Benefit                              | How It Works                                                                                                                                                                  |
| ------------------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cost Efficiency**            | Reduces upfront and ongoing IT spending.  | Shifts capital expenditure (CapEx) to a predictable, pay-as-you-go operational expenditure (OpEx) model, eliminating physical server costs.                                   |
| **Scalability & Elasticity**   | Handles fluctuating workloads seamlessly. | Automatically scales resources up or out during high-traffic spikes and shrinks them back down during quiet periods to avoid paying for idle capacity.                        |
| **Reliability & Availability** | Minimizes downtime and data loss.         | Data and workloads are mirrored across redundant, geographically separated data centers with automated failover and disaster recovery.                                        |
| **Speed & Agility**            | Fast deployment and innovation.           | Infrastructure, databases, and development environments can be provisioned in minutes with a few clicks rather than waiting weeks for physical hardware setup.                |
| **Global Reach**               | Improves international user experience.   | Allows deployment of applications across worldwide regions instantly, putting your services closer to end-users to dramatically lower latency.                                |
| **Advanced Security**          | High-tier protection and compliance.      | Leverages multi-billion dollar security investments from providers, featuring continuous monitoring, robust encryption, and built-in regulatory compliance (GDPR, ISO, etc.). |


## Hosting & Deployment Models

The foundational choices for where your infrastructure lives and who manages it.

### On-Premises (Traditional)
- **What it is:** Hosting, managing, networking, and storage are all handled on company-owned servers in a physical location.
- **Advantages:**
	- **Complete control** over data and security.
	- Predictable ongoing costs.
- **Disadvantages:**
	- **High CapEx and OpEx** (Capital and Operational Expenditures).
	- **Slow to scale** (requires buying and racking new physical hardware).
	- Requires dedicated internal resources for maintenance and upgrades.

#### Cloud Computing
- **What it is:** Servers and infrastructure are maintained by **3rd-party providers** over the internet.
- **Advantages:**
	- **Low CapEx and OpEx** due to a pay-as-you-go model.
	- **Easily scalable** on demand.
	- Providers handle all hardware maintenance and physical security.
- **Disadvantages:**
	- **Less control** over the underlying hardware.
	- Recurring costs can spiral out of control if not strictly monitored.

### Hybrid Cloud
- **What it is:** A **combination** of on-premises infrastructure and public cloud services.
- **Advantages:**
	- **Maximum flexibility**.
	- Allows you to keep highly sensitive data in-house while using the public cloud to handle high traffic spikes (cloud bursting).
- **Disadvantages:**
	- Complex to set up and difficult to manage across two different environments.


## Cloud Service Models
This dictates how much of the stack you manage versus how much the cloud provider manages.

| Model                     | What You Rent                                                   | Your Focus & Responsibility                          | Examples                       |
| ------------------------- | --------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------ |
| **IaaS** (Infrastructure) | Raw computing power, storage, and virtual machines.             | You manage the OS, applications, and data.           | AWS EC2, Google Compute Engine |
| **PaaS** (Platform)       | A pre-configured environment to build, deploy, and manage apps. | You focus strictly on developing code and your data. | Heroku, AWS Elastic Beanstalk  |
| **SaaS** (Software)       | A complete, ready-to-use application.                           | You just log in and use the software.                | Gmail, Netflix, Salesforce     |

## Compute Architecture- Server-Based vs. Serverless
A fundamental architectural choice when building applications in the cloud.

| Feature              | Server-Based                                                  | Serverless                                                  |
| -------------------- | ------------------------------------------------------------- | ----------------------------------------------------------- |
| **Infrastructure**   | Dedicated servers running 24/7.                               | Functions spin up just to do a job, then die.               |
| **Scaling**          | Manual or auto-scaling rules you configure.                   | Instant and fully automatic.                                |
| **Pricing Model**    | Pay for allocated resources (RAM/CPU) by the hour/month.      | Pay strictly per use (charged by execution time).           |
| **System Control**   | Full control over the OS and software stack.                  | Minimal to no control over the underlying OS.               |
| **State Management** | **Stateful**: Applications easily store data across sessions. | **Stateless**: Requires external databases for persistence. |
| **Execution Time**   | No restrictions on how long a process runs.                   | Subject to strict timeouts (e.g., 15 minutes).              |
