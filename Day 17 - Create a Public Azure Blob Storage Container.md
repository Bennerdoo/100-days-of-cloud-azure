# Question

As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize public Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named `datacenterst2032` and a `public` Blob container named `datacenter-blob-21177` within the storage account. Make sure anonymous read access for containers and blobs is enabled.

# Step-by-Step Solution

## Option 1: Using Azure CLI (Recommended for Terminal Environments)

### Step 1: Create the Storage Account

Create the storage account and explicitly allow public blob access at the account level:

```Bash
az storage account create \
  --name datacenterst2032 \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access true
```

(Replace `<YOUR_RESOURCE_GROUP>` with your assigned lab resource group name).

#### Step 2: Create the Public Blob Container

Create the container with anonymous read access enabled for both containers and blobs (container level):

```Bash
az storage container create \
  --account-name datacenterst2032 \
  --name datacenter-blob-21177 \
  --public-access container \
  --auth-mode login
```

Verification via CLI:

```Bash
az storage container show \
  --account-name datacenterst2032 \
  --name datacenter-blob-21177 \
  --auth-mode login \
  --query "{ContainerName:name, PublicAccess:properties.publicAccess}" \
  --output table
```

## Option 2: Using the Azure Portal

### 1. Create Storage Account:

- Search Storage Accounts.
- Log in to the Azure Portal.
- Search for `Storage accounts` in the top search bar and select it.
- Click `+ Create`.
- Fill in `Subscription` and `Resource group`.
- Set `Storage account name` to `datacenterst2032`.
- Select your lab's designated region (e.g., Central US or South Central US).
- Under the `Advanced` tab, ensure `Allow enabling anonymous access (or Allow Blob anonymous access)` is set to `Enabled`.
- Click `Review + create`, then click `Create`.

### 2. Navigate to Container Settings:

- Open `Containers` Blade.
- Click `Go to resource` to open `datacenterst2032`.
- In the left navigation menu under `Data storage`, select `Containers`.

### 3. Create Public Container:

- Provision `datacenter-blob-21177`.
- Click `+ Container` at the top action bar.
- Set `Name` to `datacenter-blob-21177`.
- Set `Anonymous access level` to `Container (anonymous read access for containers and blobs)`.
- Click `Create`.