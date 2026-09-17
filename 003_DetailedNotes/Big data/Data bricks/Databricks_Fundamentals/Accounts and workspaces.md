---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Databricks_Fundamentals/Databricks_Fundamentals|Databricks_Fundamentals]]"
tags:
  - DataBricks/fundamentals
index: 2
type: topic
---
# Accounts and workspaces

```ad-summary
collapse: true
title: Summary

## Account
- An account is created at organization level
- Admin task such as managing workspaces, metastores, users and account-level settings
  
## Workspace
- Multiple workspaces can be used to separate isolated environments
- Like for development, production or project wise workspaces
- Each workspace has resource group that cannot be shared among multiple workspaces

### Steps to Create Azure Workspace
- [ ] Step 1: Sign in to the Azure Portal
- [ ] Step 2: Initiate Workspace Creation
- [ ] Step 3: Configure the Basics
- [ ] Step 4: Configure Networking
- [ ] Step 5: Apply Tags
- [ ] Step 6: Review and Create
- [ ] Step 7: Launch the Workspace

```

## Databricks Account

* **Scope:** Usually, an organization has a single Databricks account.
* **Management:** Administrative tasks such as managing workspaces, metastores, users, and account-level settings are performed via the Databricks Account console (`[https://accounts.databricks.net/](https://accounts.databricks.net/)`).
* **Access:** This is accessible only to the account admin, not regular workspace users.

### Workspaces

* **Function:** Each workspace acts as a separate development or operational environment associated with the account.
* **Use Cases:** Multiple workspaces can be used to separate environments (like development, production, and staging) or for project/departmental separation.
* **Azure Specifics:** When Databricks is deployed in Azure, each workspace creates its own managed resource group. Databricks does not allow managed resource groups to be shared among workspaces because each workspace has its own isolated compute, network, and storage.

## Steps to Create Azure Workspace

#how-to 

Pre-requisite : [[002_topics/Big data/Cloud/How to - Create azure account and resource group|How to - Create azure account and resource group]]

Link: [How to create Azure Databricks Workspace using Azure Portal](https://www.youtube.com/watch?v=q9HyiKLWfQY)

### **Step 1: Sign in to the Azure Portal**

- Navigate to the [Azure Portal](https://www.google.com/search?q=https://portal.azure.com/) and log in using your Microsoft credentials associated with your Azure subscription.
- Create a resource in resource group
![[005_assets/Big_Data/How to/001_create_azure_workspace.jpg]]
### **Step 2: Initiate Workspace Creation**

1. In the top search bar, type **Azure Databricks** and select it from the services list.
    
2. On the Azure Databricks page, click the **+ Create** button to open the deployment wizard.
    ![[005_assets/Big_Data/How to/002_create_azure_workspace.jpg]]

### **Step 3: Configure the Basics**

Under the **Basics** tab, provide the foundational details for your workspace:
![[005_assets/Big_Data/How to/003_create_azure_workspace.jpg]]
- **Subscription:** Select the Azure subscription you want to bill this resource to.
    
- **Resource Group:** Select **Create new** and provide a descriptive name (e.g., `rg-databricks-dev`). This logical container helps manage and clean up resources later.
    
- **Workspace Name:** Enter a unique name for your Databricks workspace.
    
- **Region:** Choose the geographic region closest to you or your data sources to minimize latency.
    
- **Pricing Tier:** * **Standard:** Good for single-user or basic workloads.
    
    - **Premium:** Recommended for enterprise environments. It includes advanced features like role-based access control (RBAC) and Databricks SQL.
        
- **Managed Resource Group Name:** You can leave this blank (Azure will auto-generate one) or provide a custom name. Azure uses this locked group to manage the compute infrastructure your workspace will spin up.

### **Step 4: Configure Networking (Optional)**

Click **Next: Networking >**.
[[005_assets/Big_Data/How to/004_create_azure_workspace.jpg]]
By default, Azure deploys Databricks in a Microsoft-managed Virtual Network (VNet).

- If your organization requires strict network security or direct connection to on-premises databases, select **Yes** under **Deploy Azure Databricks workspace in your own Virtual Network (VNet)**. This is known as VNet injection.
    
- If you are just testing or learning, you can leave this set to **No**.

### **Step 5: Apply Tags (Optional)**!

Click **Next: Tags >**.

Tags are key-value pairs (e.g., `Environment` : `Development`) that help you categorize resources for billing tracking and logical organization across your Azure account.

### **Step 6: Review and Create**
![[005_assets/Big_Data/How to/005_create_azure_workspace.jpg|300]]
1. Click **Review + Create**. Azure will run a quick validation check on your configurations.
    
2. Once you see the "Validation passed" message, click **Create**.
    
3. The deployment process will begin and typically takes between 5 to 10 minutes. You can monitor the progress on the deployment screen.

### **Step 7: Launch the Workspace**
![[005_assets/Big_Data/How to/006_create_azure_workspace.jpg]]
1. When the deployment is complete, click the **Go to resource** button.
    
2. On the Azure Databricks overview page, click the **Launch Workspace** button.
    
3. A new browser tab will open, logging you into the Databricks interface via Microsoft Entra ID (formerly Azure Active Directory).
    
