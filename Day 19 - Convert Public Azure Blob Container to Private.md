# Question

The Nautilus DevOps team has been using Azure Blob Storage to manage their data. Recently, they realized that one of their containers, currently public, needs to be restricted for internal use only. Your task is to convert a public Azure Blob container to private.

Two blob containers named `xfusion-container-16713` and `xfusion-priv-1526` are available in the `centralus` region within the storage account `xfusionst17034`. The `xfusion-container-16713` is currently public, and `xfusion-priv-1526` is private.

1) Convert the blob container `xfusion-container-16713` from public to private while leaving `xfusion-priv-1526` unchanged.
2) Make sure the access level for `xfusion-container-16713` is set to `private` with no public access.

# Step-by-Step Solution

## Option 1: Using Azure CLI (Recommended for Terminal Hosts)

Run the az storage container set-permission command to set the access level to off (Private):

```bash
az storage container set-permission \
  --account-name xfusionst17034 \
  --name xfusion-container-16713 \
  --public-access off \
  --auth-mode login
```

(If using an account key instead of Azure AD login, pass `--account-key <YOUR_ACCOUNT_KEY>` instead of `--auth-mode login`).

### Verification via CLI

Check the access levels for both containers to confirm `xfusion-container-16713` is private and `xfusion-priv-1526` remains unchanged:

```bash
az storage container show \
  --account-name xfusionst17034 \
  --name xfusion-container-16713 \
  --auth-mode login \
  --query "{Container:name, PublicAccess:properties.publicAccess}" \
  --output table
```

**Expected Output**:
```
PublicAccess: null or off (indicating Private access).
```

## Option 2: Using the Azure Portal

### 1. Navigate to Storage Account:Search Storage Accounts.

Log in to the Azure Portal.
Search for `Storage accounts` in the top bar and select `xfusionst17034`.

### 2. Open Containers Blade:Locate Containers.

In the left navigation menu under `Data storage`, click `Containers`.
Locate `xfusion-container-16713` in the list.

### 3. Change Access Level to Private:Update Access Level.

Click the `...` (ellipsis) menu on the right side of the `xfusion-container-16713` row (or open the container and click `Change access level` at the top).
Select `Change access level`.
In the drop-down menu, select `Private` (no anonymous access).
Click `OK` or `Save`.