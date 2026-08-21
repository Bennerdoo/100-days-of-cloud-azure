# Question

The Nautilus DevOps team is migrating a portion of its infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team identified one VM that is experiencing increased workload demands and requires additional compute resources to maintain optimal performance.

1) Change the VM size from Standard_B1s to Standard_B2s for the virtual machine named nautilus-vm.

2) Ensure the VM is in the running state after the size change is complete.

# Step by Step Solution

## Option 1: Using Azure CLI (Recommended for Terminal Environments)

### Step 1: Find your resource group name

```bash
az group list -o table
```

### Step 2: Resize the VM

Execute the az vm resize command to change the instance size to Standard_B2s:

```Bash
az vm resize \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-vm \
  --size Standard_B2s
```

(Replace <YOUR_RESOURCE_GROUP> with the name of your assigned resource group).

### Step 3: Start/Ensure the VM is Running

Resizing an Azure VM typically causes an automatic reboot or deallocation/restart cycle. Run az vm start to guarantee it returns to the Running state:

```Bash
az vm start \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-vm
```

### Step 4: Verify Size and Power State

Confirm both the new size and power status:

```Bash
az vm get-instance-view \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-vm \
  --query "{Size:hardwareProfile.vmSize, PowerState:instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus | [0]}" \
  --output table
```

Expected Output:
```
Size: Standard_B2s
PowerState: VM running
```

## Option 2: Using the Azure Portal

### 1. Navigate to Virtual Machines:Locate the VM.

Log in to the Azure Portal.
Search for Virtual machines in the top search bar and select nautilus-vm.

### 2. Select Size Options:Change VM SKU.

In the left navigation menu under Settings, click on Size.
In the list of available sizes, search for and select Standard_B2s (2 vCPUs, 4 GiB RAM).
Click Resize at the bottom of the pane.Wait for the portal notification confirming the VM has been successfully resized.

### 3. Confirm VM Status:Verify Running State.

Go back to the Overview blade for nautilus-vm.
Check the Status field. If it displays Stopped or Deallocated, click Start at the top menu bar.
Confirm Status shows Running and Size displays Standard_B2s before submitting the task.