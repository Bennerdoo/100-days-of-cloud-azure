# Question

As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize private Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named `datacenterst29165` and a private Blob container named `datacenter-blob-9409` within the storage account.

# Step-by-Step Solution

## Option 1: Using Azure CLI (Recommended)

### 1. Create the Storage Account

Execute the `az storage account create` command:

```Bash
az storage account create \
  --name datacenterst29165 \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access false
```


(Replace <YOUR_RESOURCE_GROUP> with the name of your assigned resource group).

### 2. Create the Private Blob Container

Execute the `az storage container create` command:

```Bash
az storage container create \
  --account-name datacenterst29165 \
  --name datacenter-blob-9409 \
  --public-access off \
  --auth-mode login
```

### Verification via CLI
```Bash
az storage container show \
  --account-name datacenterst29165 \
  --name datacenter-blob-9409 \
  --auth-mode login \
  --query "{ContainerName:name, PublicAccess:properties.publicAccess}" \
  --output table
```

## Option 2: Using the Azure Portal

### 1. Create Storage Account

Search Storage Accounts.

Log in to the Azure Portal.

Search for Storage accounts in the top search bar and select it.

Click + Create.

Select your Subscription and Resource group.

Set Storage account name to datacenterst29165.

Select your lab's designated region.

Leave Primary service/Performance/Redundancy settings as default (or select Standard / LRS).

Click Review + create, then click Create. Wait for deployment to finish.

### 2. Navigate to Container Settings

Open Containers Blade.

Click Go to resource to open datacenterst29165.

In the left navigation menu under Data storage, click Containers.

### 3. Create Private Container

Provision datacenter-blob-9409.

Click + Container at the top menu bar.

Set Name to datacenter-blob-9409.

Set Anonymous access level to Private (no anonymous access).

Click Create.