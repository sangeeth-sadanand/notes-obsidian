---
up:
  - "[[003_skills/data-engineering/01 Introduction/01 Introduction|01 Introduction]]"
down:
prev:
topic: false
question: What is cloud, its types and advantages?
---
# What is cloud, its types and advantages?

> [!Summary] 
> 
> - The cloud are servers, database or software that can be accessed over internet rather than being stored locally.
> - We can have a public, private and a hybrid type of cloud based on deployment model.
> - Based on service categories we can have Infrastructure as a service, Software as service and platform as a service.
> - The main advantages of cloud is we can have easy scalability, cost efficency (less cap Ex and Op Ex based on usage).
> - High accessibility & collaboration
> - Easy disaster recovery
> 
> | **Feature**     | **Public Cloud**           | **Private Cloud**         | **Hybrid Cloud** |
> | --------------- | -------------------------- | ------------------------- | ---------------- |
> | **Cost**        | Low (Pay-per-use)          | High (Upfront investment) | Medium           |
> | **Security**    | Shared security model      | Highest (Isolated)        | Flexible         |
> | **Maintenance** | None (Handled by provider) | High (Handled by user)    | Shared           |
> | **Scalability** | Near-infinite              | Limited by hardware       | Highly flexible  |



In simple terms, the **Cloud** refers to servers, databases, and software that are accessed over the internet rather than being stored locally on your physical hard drive. 
Instead of owning and maintaining your own hardware, you "rent" computing power and storage from providers like Amazon (AWS), Microsoft (Azure), or Google (GCP).

## 1. Types of Cloud Deployment Models
There are three main ways to deploy cloud services, depending on who owns the infrastructure and how much control you need.
- **Public Cloud:** Services are delivered over the public internet and shared across multiple organizations (tenants). It is the most common model and requires no hardware maintenance from the user.
- **Private Cloud:** The cloud infrastructure is dedicated solely to one organization. It can be physically located on-site or hosted by a third-party provider, offering higher security and control.
- **Hybrid Cloud:** A "best of both worlds" approach that connects public and private clouds. It allows data and applications to be shared between them, providing greater flexibility for sensitive workloads.

## 2. Cloud Service Categories (The "As-a-Service" Stack)
Cloud computing is usually broken down into three layers, often visualized as a pyramid:
1. **Infrastructure as a Service (IaaS):** You rent the basic building blocks (servers, storage, networks). You manage the OS and apps. _Ex: AWS EC2, Google Compute Engine._
2. **Platform as a Service (PaaS):** Provides a framework for developers to build and deploy apps without worrying about the underlying infrastructure. _Ex: Heroku, Google App Engine._
3. **Software as a Service (SaaS):** Fully managed applications delivered via a web browser. _Ex: Gmail, Salesforce, Slack._

## 3. Key Advantages of the Cloud
Moving to the cloud isn't just a trend; it offers significant business and technical benefits:
- **Cost Efficiency (OpEx vs. CapEx):** You move from a "Capital Expenditure" model (buying expensive hardware upfront) to an "Operating Expenditure" model (pay-as-you-go). You only pay for the resources you actually use.
- **Scalability:** If your website suddenly gets a massive spike in traffic, you can scale up your server capacity instantly. When the traffic drops, you scale back down to save money.
- **Accessibility & Collaboration:** Since the data lives on the internet, teams can access files and collaborate in real-time from anywhere in the world.
- **Disaster Recovery:** Leading cloud providers have multiple data centers globally. If one fails, your data is usually backed up in another location, ensuring high availability.
- **Automatic Updates:** You don't have to manually update software or patch servers; the cloud provider handles the security and maintenance for you.

| **Feature**     | **Public Cloud**           | **Private Cloud**         | **Hybrid Cloud** |
| --------------- | -------------------------- | ------------------------- | ---------------- |
| **Cost**        | Low (Pay-per-use)          | High (Upfront investment) | Medium           |
| **Security**    | Shared security model      | Highest (Isolated)        | Flexible         |
| **Maintenance** | None (Handled by provider) | High (Handled by user)    | Shared           |
| **Scalability** | Near-infinite              | Limited by hardware       | Highly flexible  |

