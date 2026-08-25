# Question

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.


Create a managed disk with the following requirements:

- Name of the disk should be datacenter-disk.
- Disk type must be Standard_LRS.
- Disk size must be 2 GiB.



# Step-by-Step Solution

## Option 1: Using the Azure Portal

### Step 1: Navigate to Disks Service:

- Log in to the Azure Portal (portal.azure.com).
- In the top search bar, type Disks and select Disks under Services.
- Click + Create (or + Add).

### Step 2: Select Subscription and Resource Group:

- Select your target Subscription.
- Choose an existing Resource Group (or click Create new if required by your environment).

### Step 3: Configure Disk Name, Type, and Size:

- In the Disk name field, enter datacenter-disk.
- Select your target Region (e.g., East US).
- Under Redundancy, select Locally-redundant storage (LRS).
- Next to Size, click Change size.
- In the size selection window:
- Set Storage type to Standard HDD (LRS) or Standard SSD (LRS) (Standard_LRS).
- Set Custom disk size (GiB) to 2.
- Click OK.

### Step 4: Deploy the Disk

- Review & Create.
- Click Review + create at the bottom of the screen.
- Once validation passes, click Create.

## Option 2: Using the Azure CLI
Run the following command in Azure Cloud Shell or your local CLI to create the disk directly:

```bash
az disk create \
  --resource-group <your-resource-group> \
  --name datacenter-disk \
  --sku Standard_LRS \
  --size-gb 2
```

### Verification

To verify that the disk was created with the correct specifications:

- **Azure Portal**: Go to Disks, select datacenter-disk, and verify under Size / OS disk performance:Size: 2 GiBStorage type / Redundancy: Standard LRSAzure CLI:

```bash
az disk show \
  --resource-group <your-resource-group> \
  --name datacenter-disk \
  --query "[name, sku.name, diskSizeGb]" \
  --output table
```

The returned output will confirm datacenter-disk, Standard_LRS, and 2.