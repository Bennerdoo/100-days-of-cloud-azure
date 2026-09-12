# Question

The Nautilus DevOps Team has received a request from the Networking Team to set up a new public VNet to support a set of public-facing services. This VNet will host various resources that need to be accessible over the internet. As part of this setup, you need to ensure the VNet has public subnets with automatic public IP assignment for resources. Additionally, a new VM will be launched within this VNet to host public applications that require SSH access. This setup will enable the Networking Team to deploy and manage public-facing applications.
Create a public VNet named `nautilus-pub-vnet`, and a subnet named `nautilus-pub-subnet` under the same, make sure public IP is being auto-assigned to resources under this subnet. Further, create a VM named `nautilus-pub-vm` under this VNet. Make sure SSH port 22 is open for this instance and accessible over the internet. Use the Azure portal to complete the task and ensure that SSH access is configured correctly.

# Step by Step Solution

## Option 1: using azure console

### Step 1. Create nautilus-pub-vnet and Subnet:


1.Provision Virtual Network.

Sign in to the Azure Portal.Search for Virtual networks in the top search bar and click + Create.

In the Basics tab:

Resource group: Select your assigned default resource group.
Name: Enter nautilus-pub-vnet.
Region: Select your lab's designated region (e.g., East US or South Central US).
Select the IP addresses tab:

Set or accept the address space (e.g., 10.0.0.0/16).
Under Subnets, click + Add subnet (or edit default).
Set Subnet name to nautilus-pub-subnet.
Set Subnet address range (e.g., 10.0.1.0/24).
Click Add.

Click Review + create, then click Create.



### Step2. Create nautilus-pub-vm with Public IP:Deploy VM in the public subnet.

Search for Virtual machines in the search bar and click + Create > Azure virtual machine.

In the Basics tab:

Resource group: Select the same resource group used for the VNet.

Virtual machine name: Enter `nautilus-pub-vm`.

Region: Select the same region as `nautilus-pub-vnet`.

Image: Select Ubuntu Server 22.04 LTS - x64 Gen2 (or any available Ubuntu image).

Size: Select Standard_B1s (or default lab size).

Authentication type: Choose SSH public key (or Password).

Username: Enter azureuser.

Inbound port rules: Under Public inbound ports, select Allow selected ports, then check SSH (22).

Select the Networking tab:

Virtual network: Select `nautilus-pub-vnet`.

Subnet: Select `nautilus-pub-subnet`.

Public IP: Select Create new (or accept the auto-generated public IP, e.g., `nautilus-pub-vm-ip`) to ensure an external IP address is dynamically/statically assigned.

NIC network security group: Choose Basic or Advanced and verify that an inbound rule for SSH (Port 22) from 0.0.0.0/0 is enabled.

Click Review + create, then click Create.

### Step 3. Verify SSH Access over the Internet:Confirm SSH accessibility.

Once deployment finishes, click Go to resource to view the `nautilus-pub-vm` overview page.

Copy the Public IP address listed in the top right summary card.

Open a terminal on your client/landing host and verify SSH connectivity:

```bash
ssh azureuser@<VM_PUBLIC_IP>
```

Ensure the SSH login prompt connects successfully without timing out.

## Option 2: Azure CLI Commands

```bash
# Get current resource group
RG=$(az group list --query "[0].name" -o tsv)
LOCATION=$(az group list --query "[0].location" -o tsv)

# Create public VNet and Subnet
az network vnet create \
  --resource-group $RG \
  --name nautilus-pub-vnet \
  --location $LOCATION \
  --address-prefix 10.0.0.0/16 \
  --subnet-name nautilus-pub-subnet \
  --subnet-prefix 10.0.1.0/24

# Create VM with attached Public IP and open SSH port 22
az vm create \
  --resource-group $RG \
  --name nautilus-pub-vm \
  --vnet-name nautilus-pub-vnet \
  --subnet nautilus-pub-subnet \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --storage-sku Standard_LRS \
  --public-ip-address nautilus-pub-pip \
  --public-ip-sku Standard \
  --admin-username azureuser \
  --generate-ssh-keys \
  --nsg-rule SSH