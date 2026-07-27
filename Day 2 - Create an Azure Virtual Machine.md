# Question

The Nautilus DevOps team is planning to migrate a portion of their infrastructure to the Azure cloud incrementally. As part of this migration, you are tasked with creating an Azure Virtual Machine (VM).

The requirements are:

1) Use the existing resource group.

2) The VM name must be nautilus-vm, it should be in southcentralus region.

3) Use the Ubuntu 24.04 LTS image for the VM.

4) The VM size must be Standard_B1s.

5) Attach a default Network Security Group (NSG) that allows inbound SSH (port 22).

6) Attach a 30 GB storage disk of type Standard HDD.

7) The rest of the configurations should remain as default.

After completing these steps, make sure you can SSH into the virtual machine.

## Step by step solution

### Step 1: Navigate to Virtual Machine Creation
Sign in to the Azure Portal using your provided credentials.

In the top search bar, type Virtual machines and select Virtual machines from the search results.

Click + Create at the top-left and select Azure virtual machine.

### Step 2: Configure the Basic Settings
In the **Basics** tab, fill in the settings as follows:

| Setting                  | Selection / Value |
| ------------------------ | ------------------- |
| Subscription             | Select your default/provided Azure Subscription. |
| Resource group           | Select the existing resource group specified for your lab. |
| Virtual machine name     | nautilus-vm         |
| Region                   | (US) South Central US |
| Availability options     | No infrastructure redundancy required (default) |
| Security type            | Standard or Trusted launch (default) |
| Image                    | Select Ubuntu Server 24.04 LTS - x64 Gen2 
| VM size                  | "Click See all sizes, search for and select Standard_B1s (1 vCPU, 1 GiB RAM)." |

![day2 image1](./images/day2_image1.png)
![day2 image2](./images/day2_image2.png)

Click **Review + create** at the bottom of the page.

Inbound Port Rules:
**Public inbound ports:** Select Allow selected ports.
**Select inbound ports:** Choose SSH (22).  

### Step 3: Configure Disks
Click Next: Disks > at the bottom of the screen.
**Under OS disk size:** select Custom disk size (GiB) and enter 30 (or select 30 GiB if available in the drop-down).
**Under OS disk type:** select Standard HDD (locally-redundant storage).
Leave all other disk settings as default.

![day2 image3](./images/day2_image3.png)

### Step 4: Configure Networking
**Click Next:** Networking >
**Virtual network / Subnet:** Azure will automatically select or create the default network configuration for the resource group.
**Public IP:** Ensure a Public IP is being created/assigned (default behavior).
**NIC network security group:** Ensure Basic is selected.
**Inbound ports:** Ensure Allow selected ports is selected with SSH (22) checked.

### Step 5: Review + Create
Click Review + create at the bottom of the page.
Azure will run a validation check. Once you see Validation passed, double-check your specs:
 **Name:** nautilus-vm
 **Region:** southcentralus
 **Image:** Ubuntu 24.04 LTS  
 **Size:** Standard_B1s
 **Disk:** 30 GB Standard HDDClick Create.
Wait a few minutes for deployment to complete, then click Go to resource.

![day2 image4](./images/day2_image4.png)

### Step 6: Connect via SSH
Once the deployment completes and you are on the virtual machine's Overview page, locate the **Public IP address**.
Open your local terminal or SSH client (like PuTTY or Windows Terminal).
Use the following command, replacing `<public-ip-address>` with the IP address you noted:
```bash
ssh azureuser@<public-ip-address>
```
If prompted to accept a fingerprint, type yes and press Enter.
Enter your password when asked.
You should now be logged into your Azure virtual machine.
