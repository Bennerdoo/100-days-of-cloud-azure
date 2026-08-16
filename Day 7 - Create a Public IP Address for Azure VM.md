# Question

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
For this task, allocate a **Public IP** address, name it as **devops-pip**.

# Step-by-Step Solution

## Option 1: Using Azure CLI

### Find your resource group name

```bash
az group list -o table
```


### Run the following command to allocate a static Public IP:

```bash
az network public-ip create \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name devops-pip \
  --allocation-method Static \
  --sku Standard
```

(Replace `<YOUR_RESOURCE_GROUP>` with the existing resource group assigned to your lab).

## Option 2: Using the Azure Portal

### 1. Navigate to Public IP Addresses

- Search for **Public IP resource**.
- Log in to the Azure Portal.
- In the top search bar, search for **Public IP addresses** and select it.
- Click **+ Create**.


### 2. Configure Public IP Settings

Fill in the fields as follows:

| Setting | Value / Selection |
|--------|---------------------|
| Subscription | Select your active/lab subscription |
| Resource group | Select your existing lab resource group |
| Name | devops-pip |
| Region | Select your designated region (e.g., Central US or South Central US) |
| SKU | Standard (default) |
| Tier | Regional (default) |
| IP Version | IPv4 |
| Allocation | Static |


### 3. Review + Create

- Click **Review + create** at the bottom of the page.
- Once Validation passed appears, click **Create**.

## Verification

To verify the Public IP has been provisioned:

```bash
az network public-ip show --resource-group <YOUR_RESOURCE_GROUP> --name devops-pip --query "{Name:name, IP:ipAddress, ProvisioningState:provisioningState}" --output table
```