# Question

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a network security group (NSG) with the following requirements:

- Name of the NSG should be `nautilus-nsg`.
- Add an inbound security rule named `Allow-HTTP` for `HTTP` service on port `80`, with the source CIDR range of `0.0.0.0/0`.
- Add another inbound security rule named `Allow-SSH` for `SSH` service on port `22`, with the source CIDR range of `0.0.0.0/0`.

# Step by step solutions

## Option 1: Using Azure CLI (Recommended for Azure Cloud Shell in the lab)

### Step 1: Create the Network Security Group

First, find your resource group name:

```bash
az group list --output table
```

Then:

```bash
az network nsg create \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --name nautilus-nsg
```

(Replace <YOUR_RESOURCE_GROUP> with the name of your assigned resource group).

### Step 2: Add the Allow-HTTP Inbound Rule (Port 80)

Bash
```bash
az network nsg rule create \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --nsg-name nautilus-nsg \
  --name Allow-HTTP \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefixes "0.0.0.0/0" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow
```

### Step 3: Add the Allow-SSH Inbound Rule (Port 22)

```bash
az network nsg rule create \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --nsg-name nautilus-nsg \
  --name Allow-SSH \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --source-address-prefixes "0.0.0.0/0" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 \
  --access Allow
```

### Verification

```bash
az network nsg rule list \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --nsg-name nautilus-nsg \
  --output table
```

## Option 2: Using the Azure Portal

### Step 1: Create Network Security Group

- Search for `NSG` in the Azure Portal.
- Log in to the Azure Portal.
- Search for `Network security groups` in the top search bar and select it.
- Click `+ Create` (or `+ Add`).
- Select your Subscription and Resource group.
- Set `Name` to `nautilus-nsg`.
- Select your lab's designated region.
- Click `Review + create`, then click `Create`.

### Step 2: Configure Allow-HTTP Rule

- Add `HTTP` inbound rule.
- Open the created `nautilus-nsg` resource.
- Under `Settings` in the left navigation menu, click `Inbound security rules`.
- Click `+ Add` at the top bar and enter the following:
  - `Source`: `Any` (or `IP Addresses` with `0.0.0.0/0`)
  - `Source port ranges`: `*`
  - `Destination`: `Any`
  - `Service`: `HTTP` (or `Custom` port `80`)
  - `Destination port ranges`: `80`
  - `Protocol`: `TCP`
  - `Action`: `Allow`
  - `Priority`: `100`
  - `Name`: `Allow-HTTP`
- Click `Add`.

### Step 3: Configure Allow-SSH Rule

- Add `SSH` inbound rule.
- Click `+ Add` again for the second rule:
  - `Source`: `Any` (or `IP Addresses` with `0.0.0.0/0`)
  - `Source port ranges`: `*`
  - `Destination`: `Any`
  - `Service`: `SSH` (or `Custom` port `22`)
  - `Destination port ranges`: `22`
  - `Protocol`: `TCP`
  - `Action`: `Allow`
  - `Priority`: `110`
  - `Name`: `Allow-SSH`
- Click `Add`.