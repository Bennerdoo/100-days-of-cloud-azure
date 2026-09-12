# Question

The Nautilus DevOps team needs to expand the storage capacity of an existing virtual machine and add an additional data disk to support increased workloads. This task requires resizing the existing VM disk and mounting a new data disk to the VM.
As a member of the team, perform the following steps:
1) Expand the existing VM xfusion-vm disk from 32Gi to 64Gi.
2) Also create a new standard HDD data disk named xfusion-disk of 64Gi and mount the disk to VM xfusion-vm at location /mnt/xfusion-disk.

## Step-by-Step Solution

### Step 1: Set Up Environment Variables
Retrieve the resource group assigned to xfusion-vm:
```bash
RESOURCE_GROUP=$(az vm list --query "[?name=='xfusion-vm'].resourceGroup" -o tsv)
LOCATION=$(az vm show --resource-group $RESOURCE_GROUP --name xfusion-vm --query "location" -o tsv)
echo "Resource Group: $RESOURCE_GROUP | Location: $LOCATION"
```

### Step 2: Expand Existing VM Disk from 32 GiB to 64 GiB
To resize the OS disk, the VM must first be deallocated.1.Stop xfusion-vm:Deallocate instance.Deallocate the VM to release compute resources for disk resizing:
```bash
az vm deallocate --resource-group $RESOURCE_GROUP --name xfusion-vm
```

2.Expand  OS Disk to 64 GiB:Resize disk resource.Retrieve the OS disk name and update its size to 64 GiB:
```bash
OS_DISK_NAME=$(az vm show --resource-group $RESOURCE_GROUP --name xfusion-vm --query "storageProfile.osDisk.name" -o tsv)

az disk update \
  --resource-group $RESOURCE_GROUP \
  --name $OS_DISK_NAME \
  --size-gb 64
```

### Step 3: Create & Attach New 64 GiB Data Disk (xfusion-disk)
Attach a new Standard HDD data disk named xfusion-disk to xfusion-vm:
```bash
az vm disk attach \
  --resource-group $RESOURCE_GROUP \
  --vm-name xfusion-vm \
  --name xfusion-disk \
  --size-gb 64 \
  --sku Standard_LRS \
  --new

```

### Step 4: Mount the New Disk on the VM
1.SSH into xfusion-vm:Retrieve Public IP.Get the VM's public IP address and log in via SSH:
```bash
PUBLIC_IP=$(az vm list-ip-addresses --resource-group $RESOURCE_GROUP --name xfusion-vm --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

ssh azureuser@$PUBLIC_IP
```

2.Format and Mount /mnt/xfusion-disk:Format & mount filesystem.Inside the xfusion-vm terminal, identify the new unformatted block device (typically /dev/sdc or /dev/sdb), format it with ext4, and mount it:
```bash
# Identify the new disk device name
lsblk

# Format the disk (assuming /dev/sdc - adjust if needed based on lsblk)
sudo mkfs.ext4 /dev/sdc

# Create mount point directory
sudo mkdir -p /mnt/xfusion-disk

# Mount the disk
sudo mount /dev/sdc /mnt/xfusion-disk

# Configure persistent mount in fstab
UUID=$(sudo blkid -s UUID -o value /dev/sdc)
echo "UUID=$UUID /mnt/xfusion-disk ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
```

### Step 5: Mount the New Disk on the VM
1.SSH into xfusion-vm:Retrieve Public IP.Get the VM's public IP address and log in via SSH:
```bash
PUBLIC_IP=$(az vm list-ip-addresses --resource-group $RESOURCE_GROUP --name xfusion-vm --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

ssh azureuser@$PUBLIC_IP
```

2.Format and Mount /mnt/xfusion-disk:Format & mount filesystem.Inside the xfusion-vm terminal, identify the new unformatted block device (typically /dev/sdc or /dev/sdb), format it with ext4, and mount it:
```bash
# Identify the new disk device name
lsblk

# Format the disk (assuming /dev/sdc - adjust if needed based on lsblk)
sudo mkfs.ext4 /dev/sdc

# Create mount point directory
sudo mkdir -p /mnt/xfusion-disk

# Mount the disk
sudo mount /dev/sdc /mnt/xfusion-disk

# Configure persistent mount in fstab
UUID=$(sudo blkid -s UUID -o value /dev/sdc)
echo "UUID=$UUID /mnt/xfusion-disk ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
```

3.Expand OS Partition inside Guest OS:Grow OS filesystem.Expand the OS partition (/dev/sda1 or /dev/root) to consume the newly resized 64 GiB OS disk space:
```bash
sudo growpart /dev/sda 1
sudo resize2fs /dev/sda1
```

### Step 5: VerificationVerify that both the resized OS disk and the new data disk are correctly mounted and reporting ~64 GiB:Bashdf -h / /mnt/xfusion-disk
Expected Output:PlaintextFilesystem      Size  Used Avail Use% Mounted on
/dev/sda1        63G  ...   ...  ...  /
/dev/sdc         63G  ...   ...  ...  /mnt/xfusion-disk