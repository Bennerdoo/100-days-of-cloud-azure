# Question

## Create SSH Key Pair for Azure Virtual Machine

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create an SSH key pair with the following requirements:

- The name of the SSH key pair should be `nautilus-kp`.
- The key pair type must be `rsa`.

# Solution

## Step-by-step solution

1. First , retrieve your lab's resource group.
   ```bash
   az group list --output table
   ```

2. Run the Azure CLI command. The Azure commandline is already provided in the lab.
  > your key apir name e.g nautilus-kp in this lab"
   > your resource group name from step 1"

   ```bash
   az sshkey create \
     --name <your keypair name> \
     --resource-group <your resource group name> 
    
   ```

3. Verify the creation of the SSH key resource.

   ```bash
   az sshkey show --name "your-keypair-name" --resource-group "your-resource-group"
   ```

>