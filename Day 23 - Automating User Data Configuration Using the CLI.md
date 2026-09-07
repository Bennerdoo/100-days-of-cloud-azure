# Question

The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.
As a member of the Nautilus DevOps Team, your task is to create a VM using Azure CLI with the following specifications:
Instance Name: The VM must be named `datacenter-vm`.
Image: Use any available Ubuntu image to create this VM.
Custom Script Extension/User Data: Configure the VM to run a custom script during its launch. This script should:
Install the Nginx package.
Start the Nginx service.
Network Security Group (NSG): Ensure that the VM allows HTTP traffic on port `80` from the internet.
Instructions:
Use Azure CLI commands to set up the VM in the specified configuration.
Ensure the VM is accessible from the internet on port 80.
The Nginx service should be running after setup.


Use the Azure CLI commands to complete the task.

Notes:
Create the resources only in the `East US` region.
You may use the default resource group.

# Step by Step Solution

### Step 1: Set Up Environment & Detect Resource Group
Set the region to eastus and capture your assigned default resource group:
```bash
LOCATION="eastus"
RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)
echo "Resource Group: $RESOURCE_GROUP | Region: $LOCATION"
```

### Step 2: Prepare the User Data Script
Create a cloud-init initialization script to automatically install and start Nginx during VM creation:
```bash
cat <<'EOF' > /tmp/custom_data.txt
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
EOF
```

### Step 3: Deploy VM & Open Port 80
1. Deploy datacenter-vm in East US:

Passes cloud-init script & generates keys.

Run az vm create passing --custom-data to execute the Nginx installation script on launch:

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --storage-sku Standard_LRS \
  --location $LOCATION \
  --admin-username azureuser \
  --custom-data /tmp/custom_data.txt \
  --generate-ssh-keys
  
```

2. Allow Inbound HTTP Traffic (Port 80):

Updates NSG rule.

Open port 80 on the Network Security Group associated with datacenter-vm:

```bash
az vm open-port \
  --resource-group $RESOURCE_GROUP \
  --name datacenter-vm \
  --port 80 \
  --priority 100

```

3. Retrieve Assigned Public IP:

Fetch Public IP address.

Get the public IP address generated for datacenter-vm:

```bash
PUBLIC_IP=$(az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name datacenter-vm \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  -o tsv)

  ```

### Step 4: Verification
Test whether Nginx is installed, active, and serving HTTP traffic:
```bash
curl -I http://$PUBLIC_IP
```
Expected Output:
```
HTTP/1.1 200 OK
Server: nginx/...
...
```