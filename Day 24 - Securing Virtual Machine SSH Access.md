# Question

The Nautilus DevOps team needs to set up a new Virtual Machine (VM) on the Azure cloud that can be accessed securely from their landing host (azure-client). Follow the steps below to complete this task:
1. Create an SSH Key: On the azure-client host, check if an SSH key already exists. If it doesn’t exist, create a new SSH key on the azure-client host that will be used for password-less SSH access.
2. Create a Virtual Machine: Use the Azure Portal or Azure CLI to create a new Virtual Machine named `datacenter-vm` in the westus region. Set the VM size to `Standard_B1s` and configure the VM with SSH access for the `azureuser` account using the newly created SSH key.
3. Configure SSH Access: Ensure that the SSH key from the azure-client host is added to the azureuser account on datacenter-vm, enabling secure, password-less SSH access from the azure-client host.
4. Verify Connectivity: Test the connection from azure-client to datacenter-vm using SSH to confirm that password-less access has been set up correctly.

# Step By Step Solution

### Step 1: Detect or Create SSH Key on azure-client
Check for an existing SSH key pair, or generate a new key if one does not exist:

```bash
if [ ! -f ~/.ssh/id_rsa.pub ]; then
  ssh-keygen -t rsa -b 2048 -N "" -f ~/.ssh/id_rsa
fi
```
```bash
cat ~/.ssh/id_rsa.pub
```

### Step 2: Get Default Resource Group

Retrieve your assigned default lab resource group name:

```bash
LOCATION="westus"
RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)

# If no resource group exists, create one:
if [ -z "$RESOURCE_GROUP" ]; then
  RESOURCE_GROUP="nautilus-rg"
  az group create --name $RESOURCE_GROUP --location $LOCATION
fi

echo "Resource Group: $RESOURCE_GROUP | Region: $LOCATION"
```

### Step 3: Deploy datacenter-vm and Configure SSH Access

1.Create datacenter-vm in westus:Associates local SSH key & provisions instance.

Deploy the VM using az vm create and attach your SSH key located at ~/.ssh/id_rsa.pub:

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --storage-sku Standard_LRS \
  --location $LOCATION \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub
```

2.Get Public IP Address:Retrieve assigned Public IP.

Query the public IP address assigned to datacenter-vm:

```bash
PUBLIC_IP=$(az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name datacenter-vm \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  -o tsv)

echo "Datacenter VM Public IP: $PUBLIC_IP"
```

### Step 3.3: Verify SSH Connectivity

Connect to datacenter-vm as azureuser from azure-client:

```bash
ssh -o StrictHostKeyChecking=no azureuser@$PUBLIC_IP
```

### Step 4: Verification

Once connected via SSH, verify the login and user context:

```bash
whoami && hostname
```

Expected Output:

```plaintext
azureuser
datacenter-vm
```