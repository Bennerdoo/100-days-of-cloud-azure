# Question

The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is not tagged properly so they decided to tag it as needed.
Add the tag `Environment=dev` to the virtual machine named `devops-vm`.

# Step by Step Solution

## Option 1: Using Azure CLI (Recommended)

Run the `az vm update` command with the `--set` or `--tags` parameter:

### 1. Get the Resource Group Name

```bash
az group list -o table
```

### 2. Append/Set the Tag

```bash
az vm update \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name devops-vm \
  --set tags.Environment=dev
```

(Replace `<YOUR_RESOURCE_GROUP>` with the name of your assigned resource group).

### Verification via CLI
```bash
az vm show \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name devops-vm \
  --query "tags" \
  --output json
```

## Option 2: Using the Azure Portal

### 1. Navigate to Virtual Machines
Find devops-vm.Log in to the Azure Portal.Search for Virtual machines in the top search bar and select devops-vm.

### 2. Open Tags Section
Configure Tag Key and Value.In the left navigation menu under Display / Properties (or from the top action bar), click Tags.Under Name, enter Environment.Under Value, enter dev.Click Apply at the bottom of the pane.

### 3. Verify Tags
Confirm Tag update.Return to the Overview page for devops-vm.Check the Tags section in the upper summary area to confirm Environment : dev is listed.

