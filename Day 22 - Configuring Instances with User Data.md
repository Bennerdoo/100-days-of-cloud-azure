# Question

The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.
As a member of the Nautilus DevOps Team, your task is to create a VM with the following specifications:
Instance Name: The VM must be named devops-vm.
Image: Use any available Ubuntu image to create this VM.
Custom Script Extension/User Data: Configure the VM to run a custom script during its launch. This script should:
Install the Nginx package.
Start the Nginx service.
Network Security Group (NSG): Ensure that the VM allows HTTP traffic on port 80 from the internet.


# Step-by-Step Solution

## Step 1: Detect Resource Group & RegionAutomatically capture your assigned resource group and location:
```bash
RESOURCE_GROUP=$(az group list --query "[0].name" -o tsv)
LOCATION=$(az group list --query "[0].location" -o tsv)
echo "Resource Group: $RESOURCE_GROUP | Region: $LOCATION"
```

## Step 2: Prepare the User Data ScriptCreate a cloud-init initialization script to install and start Nginx on launch:Bash
```bash
cat <<'EOF' > /tmp/custom_data.txt
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
EOF
```

## Step 3: Create the Virtual Machine & Open HTTP Port
1.Deploy devops-vm:Passes cloud-init script & generates keys.Execute az vm create passing --custom-data to run the Nginx installation script during VM initialization:
```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name devops-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --storage-sku Standard_LRS \
  --custom-data /tmp/custom_data.txt \
  --generate-ssh-keys
```
2.Allow HTTP Inbound Traffic (Port 80):Updates NSG rule.Open port 80 on the Network Security Group associated with devops-vm:
```bash
az vm open-port \
  --resource-group $RESOURCE_GROUP \
  --name devops-vm \
  --port 80 \
  --priority 100
```
3.Get Public IP:Fetch Public IP address.Retrieve the public IP assigned to devops-vm:
```bash
PUBLIC_IP=$(az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name devops-vm \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  -o tsv)
echo "Web Server Public IP: $PUBLIC_IP"
```

## Step 4: Verification
Test whether Nginx is installed, active, and accessible over port 80:Bash
```bash
curl -I http://$PUBLIC_IP
```
Expected Output:Plaintext
```
HTTP/1.1 200 OK
Server: nginx/...
...
```