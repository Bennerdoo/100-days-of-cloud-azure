# Question

You are tasked with modifying an ARM template for deploying a virtual network. The current template is located in the `/root/arm-templates` directory under the filename `vnet-deployment-template.json`. You need to make the following changes to the template:

1) Change the name and `displayName` tag of the virtual network to `arm-vnet-devops`.
2) Update the `addressPrefixes` to `192.168.0.0/16`.
3) Add one more tag named `Environment` with value `KKE-devops`.

After making these changes, you need to deploy the ARM template using the Azure CLI.
Use the following command to find out the resource group to use:

```bash
az group list --query '[].name' --output table | grep 'kml'
```

# Step-by-Step Solution

## Step 1: Identify the Resource Group Name

Find the target resource group matching kml:

```bash
az group list --query '[].name' --output table | grep 'kml'
```

(Note down the returned resource group name, referred to as <RESOURCE_GROUP_NAME> below).

## Step 2: Edit the ARM Template

Open `/root/arm-templates/vnet-deployment-template.json` using your preferred editor (e.g., nano or vim):

```bash
nano /root/arm-templates/vnet-deployment-template.json
```

Locate the `Microsoft.Network/virtualNetworks` resource block and update the JSON properties to match the required specifications:

Update the `name` attribute of the Virtual Network resource to `"arm-vnet-devops"`.

Under `tags`, set `"displayName": "arm-vnet-devops"` and add `"Environment": "KKE-devops"`.

Under `properties.addressSpace.addressPrefixes`, set the array item to `"192.168.0.0/16"`.

**Target Structure Example: (This is not the lab's template)**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "arm-vnet-devops",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayName": "arm-vnet-devops",
        "Environment": "KKE-devops"
      },
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "192.168.0.0/16"
          ]
        }
      }
    }
  ]
}
```

Save and close the file (Ctrl+O, Enter, Ctrl+X in nano).

## Step 3: Deploy the ARM Template

1. **Run Azure CLI Deployment**: Validate template structure. Execute `az deployment group create` to deploy the modified ARM template:

```bash
az deployment group create \
  --resource-group <RESOURCE_GROUP_NAME> \
  --template-file /root/arm-templates/vnet-deployment-template.json
```

2. **Confirm Deployment Status**: Verify VNet creation and tags. Verify that `arm-vnet-devops` is deployed with the updated address space and tags:

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name arm-vnet-devops \
  --query "{Name:name, AddressSpace:addressSpace.addressPrefixes, Tags:tags}" \
  --output json