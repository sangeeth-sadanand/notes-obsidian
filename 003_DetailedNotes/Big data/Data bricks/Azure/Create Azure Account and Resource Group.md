---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Azure/Azure|Azure]]"
tags:
  - DataBricks/Azure
  - how-to
index: 101
type: topic
---
# Create Azure Account and Resource Group

```ad-summary
collapse: true 
title: Summary
### Part 1: Create an Azure Free Account

* [ ] Navigate to the Azure Free Trial Page
* [ ] Sign In or Create a Microsoft Account
* [ ] Verify Your Identity by Phone
* [ ] Verify Your Identity by Card
* [ ] Agree to Terms and Submit

### Part 2: Create a Resource Group

* [ ] Open the Resource Groups Menu
* [ ] Initiate Creation
* [ ] Configure Project Details
* [ ] Review and Create
```

---

Here is how to get your foundation set up in Azure by creating a free account and configuring your first resource group.

## **Part 1: Create an Azure Free Account**

If you don't have a Microsoft account (like Outlook or Hotmail), the signup process will guide you to create one.

1. Navigate to the Azure Free Trial Page

	- Go to the [Azure Free Account Page](https://azure.microsoft.com/free/) and click the **Start free** button.

2. Sign In or Create a Microsoft Account:

	- Log in with an existing Microsoft email. If you don't have one, click **Create one!** and follow the prompts to register a new email address.

3. Verify Your Identity by Phone:.

	- Provide your country code and phone number. Azure will send you a text message verification code or call you to confirm you are a real person.

4. Verify Your Identity by Card:

	- You must provide credit card or debit card details.

> [!Important]
> Microsoft uses this strictly for identity verification to prevent bot sign-ups. 
> You **will not be charged** unless you explicitly upgrade to a paid subscription later. Azure places a temporary small hold (usually around $1 USD) that drops off in a few days.

5. Agree to Terms and Submit: Final Step.

	- Check the box to agree to the customer agreement and privacy statement, then click **Sign up**. Once processed, you will be redirected to the **Azure Portal**.

## Part 2: Create a Resource Group

A resource group is a logical container or folder where your Azure resources (like Databricks workspaces, virtual machines, or databases) are deployed and managed together.

1. Open the Resource Groups Menu:

Once logged into the [Azure Portal](https://portal.azure.com/), search for **Resource groups** in the top search bar and click on it from the list. Alternatively, click the **Resource groups** icon on the home dashboard.

2. Initiate Creation:

On the Resource Groups page, click the **+ Create** (or **+ New**) button in the top-left corner.

**3.Configure Project Details:**Basics Tab.

Fill out the following parameters:

- **Subscription:** Select your subscription (it will default to "Azure subscription 1" or "Free Trial").
- **Resource group:** Give it a meaningful name. A common naming convention is `rg-[project]-[environment]` (e.g., `rg-databricks-dev`).
- **Region:** Choose a geographic location close to you (e.g., _East US_ or _West Europe_). This sets the default metadata location for the group.

**4.Review and Create:**

Click **Review + create** at the bottom of the screen. Azure will quickly validate the name. Once the green "Validation passed" message appears, click **Create**.

Your resource group will deploy almost instantly. You can now use this group as the destination when following the Databricks setup steps!