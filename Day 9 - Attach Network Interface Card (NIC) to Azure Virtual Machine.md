# Question

The nautilus DevOps team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing network interface (NIC) to a virtual machine (VM).

- An existing VM named nautilus-vm and a network interface named nautilus-nic already exist in the southcentralus region.
- Attach the network interface nautilus-nic to the VM nautilus-vm.
- Ensure the NIC's status is attached before submitting the task.
- Make sure that the virtual machine initialization has been completed before submitting this task.

# Step-by-Step Solution

## Option 1: Using Azure CLI (Recommended for Terminal/Jump Host)

### 1. (Prerequisite) Stop/Deallocate VM if necessary
If attaching an additional NIC to a running VM is restricted by the instance size, deallocate it first:
```bash
az vm deallocate \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-vm
```

### 2. Attach the Network Interface
Add the NIC to the VM:
```bash
az vm nic add \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --vm-name nautilus-vm \
  --nics nautilus-nic
```

### 3. Start the VM
Bring the VM back to the Running state so initialization completes:
```bash
az vm start \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-vm
```

### 4. Verify NIC Status
Confirm that nautilus-nic is attached and provisioned:
```bash
az network nic show \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-nic \
  --query "{Name:name, AttachedVM:virtualMachine.id, ProvisioningState:provisioningState}" \
  --output table
```

## Option 2: Using the Azure Portal

### 1. Check VM Status

- Stop VM if needed.
- Open the Azure Portal and navigate to Virtual machines.
- Click on nautilus-vm.
- If the portal requires the VM to be stopped to modify network interfaces, click **Stop** at the top menu bar and wait until the status shows Stopped (deallocated).

### 2. Navigate to Networking

- **Attach NIC**.
- In the left-hand menu under **Networking**, click on **Network settings (or Networking)**.
- Click **Attach network interface** from the top action bar.
- In the drop-down list, select **nautilus-nic**.
- Click **OK** or **Save**.

### 3. Start the VM

- **Complete Initialization**.
- Return to the **Overview** page for nautilus-vm.
- Click **Start** to power on the VM.
- Wait until the status changes to **Running** and the agent/initialization completes before submitting the lab.