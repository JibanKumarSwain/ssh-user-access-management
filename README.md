# ssh-user-access-management
Secure SSH user access management using public/private key authentication without sharing the root PEM file.


# SSH Access Setup for Dev & QA Teams (Without Sharing the Root PEM Key)

## Overview

Instead of sharing the **root PEM file**, each **Developer** or **QA Engineer** creates their own SSH key pair. Only the **public key** is shared with the server administrator, while the **private key** always remains on the user's local machine.

This approach is more secure and provides individual access for each team member.

---

# Step 1: Generate an SSH Key Pair (Developer/QA Machine)

Run the following command on your local system:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Example:

```bash
ssh-keygen -t ed25519 -C "dev1@example.com"
```

You'll see output similar to:

```text
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
```

Enter a meaningful filename, for example:

```
QA-team
```

or

```
dev1_ed25519
```

Next, you'll be prompted for a passphrase:

```text
Enter passphrase (empty for no passphrase):
```

- You may enter a passphrase for additional security.
- Or simply press **Enter** twice to leave it empty.

Example output:

```text
Your identification has been saved in QA-team
Your public key has been saved in QA-team.pub
```

You will now have two files:

| File | Purpose |
|-------|----------|
| `QA-team` | Private Key (Keep Secret) |
| `QA-team.pub` | Public Key (Share with Server Admin) |

> **Important**
>
> - Never share your **private key**.
> - Only share the **.pub** file.

---

# Step 2: Configure SSH Access on the Server (EC2/Linux)

Login to the server using an account with sudo privileges.

### Switch to Root

```bash
sudo -i
```

---

## Create a New User

Replace **jiban** with the actual username.

```bash
adduser jiban
```

You'll be asked to create a password.

Example:

```
Password: ********
```

---

## Create the User's Home Directory (if required)

```bash
mkdir -p /home/jiban
```

---

## Create the SSH Directory

```bash
mkdir -p /home/jiban/.ssh
```

---

## Add the Public Key

Open the `authorized_keys` file:

```bash
nano /home/jiban/.ssh/authorized_keys
```

Paste the user's **public key**.

Example:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILA267PKfmIqtRIgtSnnE+51z2Y6zcB7vQLEX6BDAuGu dev1@example.com
```

Save the file.

> **Only paste the public key (.pub).**
>
> **Never paste the private key.**

---

## Set Correct Permissions

```bash
chown -R jiban:jiban /home/jiban/.ssh

chmod 700 /home/jiban/.ssh

chmod 600 /home/jiban/.ssh/authorized_keys
```

---

# Step 3: Connect to the Server

The developer or QA engineer can now connect using their private key.

Example:

```bash
ssh -i QA-team jiban@190.06.007.130
```

or

```bash
ssh -i dev1_ed25519 dev1@190.06.007.130
```

Where:

- `-i` = Path to the private key
- `jiban` = Linux username
- `190.06.007.130` = Server Public IP

---

# Security Best Practices

- ✅ Never share the root PEM file.
- ✅ Never share your private SSH key.
- ✅ Share only the `.pub` file with the administrator.
- ✅ Create a separate Linux user for every team member.
- ✅ Remove a user's public key from `authorized_keys` when access is no longer needed.
- ✅ Use a passphrase for your private key whenever possible.

---

# Workflow Summary

```text
Developer / QA
        │
        │ Generate SSH Key Pair
        ▼
Private Key (.ed25519) ─────────────► Keep on Local Machine
Public Key (.pub) ─────────────────► Send to Server Admin
                                         │
                                         ▼
                              Add to authorized_keys
                                         │
                                         ▼
                           User Connects Using SSH Key
```

## Result

- No need to share the root PEM file.
- Each team member has their own secure SSH access.
- Access can be granted or revoked individually.
- This follows SSH security best practices.
