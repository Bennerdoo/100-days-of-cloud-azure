# Question

The Nautilus DevOps team is presently immersed in data migrations, transferring data from on-premise storage systems to Azure Blob containers. They have recently received some data that they intend to copy to one of the Blob containers.
A Blob container named `datacenter-blob-7032` already exists in the `centralus` region under the storage account `datacenterst10406`. Copy the file `/tmp/datacenter.txt` to the Blob container `datacenter-blob-7032`.

# Step-by-Step Solution

## Option 1: Using Azure CLI (Recommended)
Run the az storage blob upload command using Azure AD authentication credentials:
```bash
az storage blob upload \
  --account-name datacenterst10406 \
  --container-name datacenter-blob-7032 \
  --name datacenter.txt \
  --file /tmp/datacenter.txt \
  --auth-mode login
```

(If using account key authentication instead of Azure AD login, pass --account-key <KEY> instead of --auth-mode login).

### Verification via CLI
```bash
az storage blob list \
  --account-name datacenterst10406 \
  --container-name datacenter-blob-7032 \
  --auth-mode login \
  --query "[?name=='datacenter.txt'].{Name:name, Size:properties.contentLength}" \
  --output table
```

## Option 2: Using AzCopy
If azcopy is installed on your terminal host, run:
```bash
azcopy copy '/tmp/datacenter.txt' 'https://datacenterst10406.blob.core.windows.net/datacenter-blob-7032/datacenter.txt'
```

## Option 3: Using the Azure Portal

### 1.Navigate to Storage Account:Search Storage Accounts.
Log in to the Azure Portal.
Search for Storage accounts and select datacenterst10406.

### 2.Navigate to Containers:Open Target Container.
In the left menu under Data storage, click Containers.
Click on datacenter-blob-7032.

### 3.Upload datacenter.txt:Upload File.
Click the Upload button at the top menu bar.In the upload pane, click the folder icon and select /tmp/datacenter.txt.Click Upload.