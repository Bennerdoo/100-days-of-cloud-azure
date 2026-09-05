# Question

The Nautilus DevOps Team has received a new request from the Development Team to set up a new Azure Virtual Machine (VM). This VM will be used to host a new application that requires a stable public IP address. To ensure that the VM has a consistent public IP, a Static Public IP address needs to be associated with it. The VM will be named `nautilus-vm`, and the Static Public IP will be named `nautilus-pip`. This setup will help the Development Team to have a reliable and consistent access point for their application.

1. Create an Azure VM named `nautilus-vm` using any available Ubuntu image, with the VM size `Standard_B1s`.
2. Generate an SSH public key on the `azure-client` host and associate it with the VM for SSH access.
3. Associate a Static Public IP address named `nautilus-pip` with this VM.
4. Ensure the VM is accessible via SSH using the generated public key.

# Step by Step Solution

## Step 1: Detect Your Lab Resource Group and Region
Set environmental variables to automatically retrieve your lab's pre-assigned resource group name and region:

```bash
RESOURCE_GROUP=$(az group list --query "[0].name" --output tsv)
LOCATION=$(az group list --query "[0].location" --output tsv)
echo "Resource Group: $RESOURCE_GROUP | Region: $LOCATION"
```

## Step 2: Provision the Static Public IP (`nautilus-pip`)
Create a static public IP address before attaching it to the VM:

```bash
az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name nautilus-pip \
  --allocation-method Static \
  --sku Standard \
  --location $LOCATION
```

## Step 3: Deploy the VM (`nautilus-vm`) and Configure SSH Access

1. Create VM with Static PIP & SSH Key:
Generate keys & deploy VM.Execute az vm create passing nautilus-pip for the public IP and --generate-ssh-keys to automatically create/associate the SSH key from your azure-client host (~/.ssh/id_rsa.pub):

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --storage-sku Standard_LRS \
  --admin-username azureuser \
  --public-ip-address nautilus-pip \
  --generate-ssh-keys
```

2. Verify Public IP Assignment:
Fetch assigned IP.Retrieve the assigned IP address of nautilus-pip:

```bash
PIP_ADDRESS=$(az network public-ip show \
  --resource-group $RESOURCE_GROUP \
  --name nautilus-pip \
  --query "ipAddress" \
  --output tsv)
echo "Static Public IP Address: $PIP_ADDRESS"
```

3. SSH into Virtual Machine:
Verify remote shell.Test passwordless SSH access into nautilus-vm:

```bash
ssh -i ~/.ssh/id_rsa azureuser@$PIP_ADDRESS
```

4. Verification:

Confirm the VM is in the Running power state and bound to nautilus-pip:

```bash
az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name nautilus-vm \
  --output table
```