# Question

The Nautilus DevOps team is working on setting up secure SSH access for their virtual machines in Azure. One of the requirements is to add the SSH public key of the root user from the Azure client host (landing host) to the `xfusion-vm` Azure VM's `authorized_keys` file. This ensures secure and password-less SSH access to the VM.

### Task Details:

1\) **VM Details**:

- The VM is named `xfusion-vm` and is running in the `westus` region. The default SSH user is `azureuser` — use this user to connect to the VM.
- You need to add the root user's SSH public key from the Azure client host to the `authorized_keys` file of the VM's root user.
- The SSH public key of the root user on the Azure client host is located at `/root/.ssh/id_rsa.pub`.

2\) **Public Key Addition**:

- Copy the public key located at `/root/.ssh/id_rsa.pub` on the Azure client host to the `authorized_keys` file of the root user on `xfusion-vm`.
- Ensure that the proper permissions for the `.ssh` folder and `authorized_keys` file are set on the VM.

3\) **Verification**:

- After adding the public key, make sure that you are able to SSH into the `xfusion-vm` VM as the `root` user from the Azure client host without needing a password.

### Important Notes:

- Ensure that the VM is up and running before attempting to SSH.
- You may need to adjust the firewall or security group rules for the VM to allow SSH access.

# Step By Step Solution

The objective is to take the root user's public SSH key from the Azure client/landing host at `/root/.ssh/id_rsa.pub` and add it to the root user's `/root/.ssh/authorized_keys` on `xfusion-vm`, then verify passwordless root SSH access.

The important detail in this environment is that Azure's default `authorized_keys` entry may contain a forced command that deliberately rejects root login. Therefore, simply appending the key is not enough; that restriction must also be removed.

### Step 1: Confirm the Azure VM is running

From the Azure client/landing host:
```bash
az vm list -d -o table
az group list -o table
```
Find:
```bash
xfusion-vm
```
Make sure its power state is:
```bash
VM running
```
If necessary:
```bash
az vm start \
  --resource-group <RESOURCE_GROUP> \
  --name xfusion-vm
```
### Step 2: Find the VM's public IP address

Run:
```bash
az vm show -d \
  --resource-group <RESOURCE_GROUP> \
  --name xfusion-vm \
  --query publicIps \
  -o tsv
```
For this environment, the IP is:

```bash
172.184.248.195
```
You can store it in a variable, example:
```bash
VM_IP=172.184.248.195
```

Verify:
```bash
echo "$VM_IP"
```
### Step 3: Verify the root user's public key on the landing host

The task specifically requires:

```bash
/root/.ssh/id_rsa.pub
```

Check that it exists:
```bash
ls -l /root/.ssh/id_rsa.pub
```

Display it:
```bash
cat /root/.ssh/id_rsa.pub
```
You should see a line similar to:
```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ... root@azure-client
```
Do not copy the private key:
```bash
/root/.ssh/id_rsa
```
Only:
```bash
/root/.ssh/id_rsa.pub
```
is required.

### Step 4: Connect to xfusion-vm as azureuser

Direct root login initially fails because Azure's default SSH configuration prevents it.

Therefore, connect using the VM's default user:
```bash
ssh azureuser@172.184.248.195
```
You should get a prompt similar to:
```bash
azureuser@xfusion-vm:~$
```
### Step 5: Inspect the existing root SSH configuration

On xfusion-vm, run:
```bash
sudo sshd -T | grep -E 'permitrootlogin|pubkeyauthentication'
```
The existing Azure configuration may show:
```bash
permitrootlogin without-password
pubkeyauthentication yes
```

without-password means root can authenticate using SSH keys, but not using a password.

So key authentication itself is allowed.

### Step 6: Inspect root's existing authorized_keys

Run:
```bash
sudo cat /root/.ssh/authorized_keys
```
In this environment, the first entry contains something like:
```bash
no-port-forwarding,no-agent-forwarding,no-X11-forwarding,command="echo 'Please login as the user \"azureuser\" rather than the user \"root\".';echo;sleep 10;exit 142" ssh-rsa ...
```

This is critical.

The `command="..."` option forces SSH to execute the message instead of giving the root user a shell.

That is why:

```bash
ssh root@172.184.248.195
```

produces:

```bash
Please login as the user "azureuser" rather than the user "root".
```

Even though root public-key authentication is technically enabled.

### Step 7: Back up the existing authorized_keys

Before modifying it:
```bash
sudo cp /root/.ssh/authorized_keys \
  /root/.ssh/authorized_keys.bak
```

Verify the backup:
```bash
sudo ls -l /root/.ssh/authorized_keys*
```

### Step 8: Create /root/.ssh with the correct permissions

Run:
```bash
sudo mkdir -p /root/.ssh
```
Set ownership:
```bash
sudo chown root:root /root/.ssh
```

Set permissions:
```bash
sudo chmod 700 /root/.ssh
```

Verify:
```bash
sudo ls -ld /root/.ssh
```
You want:
```bash
drwx------ root root /root/.ssh
```

### Step 9: Install the landing host's public key

There are two good approaches.

Recommended approach: copy the key directly

Exit the VM:
```bash
exit
```
You should be back on the Azure client/landing host.

Then execute:
```bash
cat /root/.ssh/id_rsa.pub | ssh azureuser@172.184.248.195 \
  'sudo tee /root/.ssh/authorized_keys > /dev/null'
```
This replaces the VM's root authorized_keys with the public key from the landing host.

For this particular task, this is appropriate because the existing Azure-generated key contains the forced command that prevents root login.

### Step 10: Alternatively, manually install the known key

If you prefer to remain connected to the VM, you can use:
```bash
sudo tee /root/.ssh/authorized_keys > /dev/null <<'EOF'
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDjUJzq16cLJZLkm/n1wV5UCyakDeLTkc1q+RBvIUzZ0oTfg7OTa9mjQ4VCOPnlCJ1CipSLE19w7FozrXY2UT6Ee/f3uJQIaeThs2zakBxpq/YlebBt5HFhRkZ8a8Q8x4rtIblvl3cX13DAEETNvIBEPkuYP9H8yGWbeTu7ysXkuqYuAZtMfvcRjZ6eEH582rCZdSDfSOjZTKgkBFuGPoI2zp8hOFC//A1NEkdanFefQn5McuUFLyVb2xRROqIfbLgH4zF8aeBRdmGTp5+/xh85IdRqfB+FwO52Ed+T1/3QkNFaq6WxxP8uJIgDk6eceddRPUt7BNwgL3GoVImLuJfB root@azure-client
EOF
```
The key must match exactly:
```bash
cat /root/.ssh/id_rsa.pub
```
from the landing host.

### Step 11: Set the correct authorized_keys permissions

On xfusion-vm:
```bash
sudo chmod 600 /root/.ssh/authorized_keys
sudo chown root:root /root/.ssh/authorized_keys
```

Verify:
```bash
sudo ls -l /root/.ssh/authorized_keys
```
Expected:
```bash
-rw------- root root /root/.ssh/authorized_keys
```
### Step 12: Verify that the forced command has been removed

Run:
```bash
sudo cat /root/.ssh/authorized_keys
```
The line should look like:
```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ... root@azure-client
```
It should not begin with:
```bash
no-port-forwarding
```
and should not contain:
```bash
command="echo 'Please login as the user..."
```
This is an important step in this particular Azure environment.

### Step 13: Enable root SSH key authentication

Create an SSH configuration override:
```bash
sudo tee /etc/ssh/sshd_config.d/99-root-login.conf > /dev/null <<'EOF'
PermitRootLogin prohibit-password
PubkeyAuthentication yes
EOF
```
This means:
- PermitRootLogin prohibit-password → root may log in using SSH keys but not passwords.
- PubkeyAuthentication yes → public-key authentication is enabled.

This is preferable to enabling root password authentication.

### Step 14: Check for conflicting SSH configuration

Run:
```bash
sudo grep -RniE 'PermitRootLogin|DenyUsers|AllowUsers' \
  /etc/ssh/sshd_config \
  /etc/ssh/sshd_config.d/ 2>/dev/null
```

Also check the effective configuration:
```bash
sudo sshd -T | grep -E 'permitrootlogin|pubkeyauthentication'
```
The important result should be:
```bash
permitrootlogin prohibit-password
pubkeyauthentication yes
```
If permitrootlogin reports without-password, that is also acceptable for key-based authentication.

### Step 15: Validate the SSH configuration before restarting

Never restart SSH blindly after changing sshd_config.

Run:
```bash
sudo sshd -t
```
If there is no output, the configuration is valid.
If you receive an error, fix the configuration before restarting SSH.

### Step 16: Restart the SSH service

On Ubuntu:
```bash
sudo systemctl restart ssh
```
Check its status:
```bash
sudo systemctl status ssh --no-pager
```
You want to see:
```bash
Active: active (running)
```
### Step 17: Exit the VM
```bash
exit
```
You should now be back at the Azure client/landing host.

### Step 18: Test passwordless root SSH

From the landing host, run:
```bash
ssh root@172.184.248.195
```

The SSH client should authenticate using:
```bash
/root/.ssh/id_rsa
```
because the corresponding public key:
```bash
/root/.ssh/id_rsa.pub
```
was installed in:
```bash
/root/.ssh/authorized_keys
```
on xfusion-vm.

You should get a root prompt similar to:

root@xfusion-vm:~#

There should be no password prompt.

### Step 19: Verify that you are actually root

Run:
```bash
whoami
```
Expected:
```bash
root
```

Also:
```bash
id
```
You should see:
```bash
uid=0(root) gid=0(root) groups=0(root)
```

### Step 20: Final verification

You can perform the entire verification from the landing host with:
```bash
ssh root@172.184.248.195 'whoami && id'
```
Expected output should contain:
```bash
root
uid=0(root) gid=0(root) groups=0(root)
```
You can also explicitly test that no password is being requested:
```bash
ssh -o PreferredAuthentications=publickey \
    -o PasswordAuthentication=no \
    root@172.184.248.195 'whoami'
```
Expected:
```bash
root
```
This confirms that public-key authentication alone is sufficient.

### Final state

The VM should end up with:
```bash
/root/.ssh/
├── authorized_keys
└── authorized_keys.bak
```
with permissions:
```bash
/root/.ssh                  700
/root/.ssh/authorized_keys  600
```

and ownership:
```bash
root:root
```
The authorized_keys file should contain the landing host's public key:
```bash
/root/.ssh/id_rsa.pub
```
without Azure's forced command="Please login as the user azureuser..." restriction.

The effective SSH configuration should allow key-based root authentication:
```bash
PermitRootLogin prohibit-password
PubkeyAuthentication yes
```

And the final test:
```bash
ssh root@172.184.248.195
```
should successfully provide a passwordless root SSH session.

### Short version for the actual lab

If you want the quickest reliable sequence, use:
```bash
# Landing host
VM_IP=172.184.248.195
cat /root/.ssh/id_rsa.pub
ssh azureuser@$VM_IP
```
Then on xfusion-vm:
```bash
sudo cp /root/.ssh/authorized_keys /root/.ssh/authorized_keys.bak
sudo mkdir -p /root/.ssh
sudo chmod 700 /root/.ssh
sudo tee /root/.ssh/authorized_keys > /dev/null <<'EOF'
PASTE_THE_CONTENTS_OF_/root/.ssh/id_rsa.pub_HERE
EOF

sudo chmod 600 /root/.ssh/authorized_keys
sudo chown -R root:root /root/.ssh


sudo tee /etc/ssh/sshd_config.d/99-root-login.conf > /dev/null <<'EOF'
PermitRootLogin prohibit-password
PubkeyAuthentication yes
EOF

sudo sshd -t
sudo systemctl restart ssh
exit
```
Then on the landing host:
```bash
ssh root@172.184.248.195
```
and:
```bash
whoami
```
should return:
```bash
root
```