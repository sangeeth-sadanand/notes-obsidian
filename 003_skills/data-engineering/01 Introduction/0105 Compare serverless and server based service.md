---
up:
  - "[[003_skills/data-engineering/01 Introduction/01 Introduction|01 Introduction]]"
down:
prev:
topic: false
question: Compare serverless and server based service
---
# Compare serverless and server based service

> [!summary] 
> - In serverless computing infrastructure only exist only for the seconds it takes to run the code, while in server based infrastructure is dedicated 24/7 for the service
> - Serverless everything (os, update and scaling) is managed by cloud provider, while in server based Os, updates and scaling is managed by users.
> - Manual scaling is required in serverbase while in serverless instantaneous and automatic scaling is applied
> - Cost model is based on hourly or monthly usage while in serveless pay per execution model is applied
> - Server is always running in serverbased while in serverless brief delay a function is triggered after being idle

- The primary difference between **serverless** and **server-based** (often called "Infrastructure as a Service" or IaaS) computing isn't the absence of servers—after all, code always runs on a machine somewhere. 
- Instead, the difference lies in **who manages the infrastructure** and **how you pay for it**.

## 1. Core Definitions

### Server-Based (Traditional Hosting)
In this model, you rent or own a specific "virtual machine" (VM). You are responsible for choosing the OS, patching security vulnerabilities, and ensuring the server is running. You pay for the server as long as it is "turned on," regardless of whether it's actually doing any work.
- **Analogy:** Renting a house. You pay the monthly rent whether you are sleeping in it, working in it, or if it's sitting empty.

### Serverless (Function as a Service / FaaS)

The cloud provider manages the infrastructure entirely. You simply upload a "function" (a piece of code), and the provider executes it when triggered by an event. The infrastructure only exists for the seconds it takes to run that code.
- **Analogy:** Staying in a hotel or using a ride-share. You only pay for the exact time you occupy the room or the exact miles you travel.
    

## 2. Head-to-Head Comparison

|**Feature**|**Server-Based (VMs/IaaS)**|**Serverless (FaaS)**|
|---|---|---|
|**Management**|You manage OS, updates, and scaling.|Provider manages everything; you only manage code.|
|**Scaling**|Manual or pre-configured "Auto-scaling." Slow to react.|Automatic and instantaneous based on request volume.|
|**Cost Model**|Fixed hourly/monthly rate (Pay for capacity).|Pay-per-execution (Pay for usage).|
|**"Cold Start"**|None. The server is always running.|Brief delay when a function is triggered after being idle.|
|**Control**|Full control over the environment and OS.|Limited control; you must use supported runtimes.|
|**Limits**|Limited by the hardware specs you chose.|Often has timeout limits (e.g., 15 minutes max).|


## 3. Advantages and Disadvantages

### Server-Based
- **Pros:** Total control over the software stack; no "cold start" latency; better for long-running processes (like a database or a heavy computation).
- **Cons:** Higher "operational overhead" (maintenance); you pay for "idle" time when no one is using the app.

### Serverless
- **Pros:** Zero maintenance; scales from zero to thousands of users instantly; extremely cost-effective for apps with unpredictable traffic.
- **Cons:** "Cold starts" can cause slight lag for the first user; not ideal for heavy, long-running tasks due to execution time limits; "Vendor Lock-in" (it's hard to move code from AWS Lambda to Google Cloud Functions).
    
## 4. When to Use Which?
- **Choose Server-Based if:** You have a massive, steady workload that runs 24/7, you need specific OS-level configurations, or you are running a legacy application that wasn't built for the cloud.
- **Choose Serverless if:** You are building microservices, processing data triggered by events (like file uploads), or building a new app where you don't want to worry about managing servers.
