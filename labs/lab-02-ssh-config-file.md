# Lab 02 — SSH Config File: Connecting to EC2 Made Easy

## Overview

Every time you SSH into an EC2 instance, the full command 
requires the path to your key file, your username, and the 
full public IP address of the instance. That command gets 
long and easy to mistype, and the public IP changes every 
time you stop and restart your instance, making it even 
more tedious to manage.

This lab walks through setting up an SSH config file on 
your local Mac that stores all of that information under 
a simple nickname. After completing this lab, connecting 
to your EC2 instance requires just two words:

    ssh myec2

This is standard practice for engineers managing remote 
servers regularly, and reflects how cloud infrastructure 
is accessed in real professional environments.

---

## What is an SSH Config File?

The SSH config file is a plain text file stored on your 
local machine at ~/.ssh/config. It does not live on your 
EC2 instance. It lives on your Mac and tells SSH how to 
connect to remote servers using nicknames instead of full 
connection strings.

When you type ssh myec2, your Mac reads the config file, 
finds the entry labeled myec2, and uses the stored hostname, 
username, and key file path to make the connection 
automatically.

---

## Prerequisites

- Active EC2 instance in running state on AWS
- .pem key pair file saved locally on your Mac
- Lab 01 completed: you can already SSH into EC2 manually
- Mac terminal or iTerm2

---

## Steps

### Step 1 — Create the .ssh Directory

The SSH config file must live inside a hidden folder called 
.ssh in your home directory. This folder may not exist yet 
on your Mac. Run this command to create it. The -p flag 
ensures it is only created if it does not already exist, 
so running it multiple times causes no issues:

    mkdir -p ~/.ssh

Set the correct permissions on the folder. SSH requires 
this directory to be accessible only by you:

    chmod 700 ~/.ssh

![mkdir and chmod commands](screenshots/lab-02-mkdir-chmod.png)

---

### Step 2 — Create the SSH Config File

Open the config file in nano. If the file does not exist 
yet, nano creates it automatically:

    nano ~/.ssh/config

![nano opening the config file](screenshots/lab-02-nano-config.png)

---

### Step 3 — Add Your EC2 Instance Entry

Type the following into the nano editor, replacing 
YOUR-EC2-PUBLIC-IP with the current public IPv4 address 
of your EC2 instance. Find this in the AWS Console under 
EC2 → Instances → select your instance → Public IPv4 address.

    Host myec2
        HostName YOUR-EC2-PUBLIC-IP
        User ec2-user
        IdentityFile ~/Desktop/CLOUDENGINEER/brianne-aws-key-pair.pem

Each line explained:

- Host myec2 — the nickname you will type to connect
- HostName — the public IP address of your EC2 instance
- User — the default username for Amazon Linux instances
- IdentityFile — the full path to your .pem key file on 
  your local Mac

Save and exit nano: Ctrl + O → Enter → Ctrl + X

![Config file contents in nano](screenshots/lab-02-config-contents.png)

---

### Step 4 — Set Permissions on the Config File

SSH requires the config file to have restricted permissions. 
Run this to set them correctly:

    chmod 600 ~/.ssh/config

chmod 600 means only you can read and write the file. 
No one else on the system can access it. SSH will refuse 
to use a config file with looser permissions as a security 
enforcement — the same reason it requires chmod 400 on 
your .pem key file.

![chmod 600 on config file](screenshots/lab-02-chmod-config.png)

---

### Step 5 — Verify the Config File Contents

Confirm the file was saved correctly:

    cat ~/.ssh/config

![cat config file output](screenshots/lab-02-cat-config.png)

---

### Step 6 — Connect Using the Nickname

    ssh myec2

You should see the Amazon Linux 2023 banner and your 
ec2-user prompt — connected with just two words.

![Successful ssh myec2 connection](screenshots/lab-02-ssh-myec2-success.png)

---

## Updating the IP Address When It Changes

Every time you stop and restart your EC2 instance, AWS 
assigns it a new public IP address. When this happens, 
update your config file with the new IP:

    nano ~/.ssh/config

Change the HostName line to the new IP address, save, 
and exit. The ssh myec2 command will work immediately 
with no other changes needed.

---

## Troubleshooting

**Issue — SSH config file not found after creation**

After saving the config file in nano and typing ssh myec2, 
I received this error:

    ssh: Could not resolve hostname myec2: nodename nor 
    servname provided, or not known

I discovered that the ~/.ssh/ directory did not exist yet 
on my Mac, which meant the config file had nowhere to be 
saved. The fix was to create the directory first before 
attempting to create the config file inside it.

Commands that resolved the issue:

    mkdir -p ~/.ssh
    chmod 700 ~/.ssh

The -p flag on mkdir is important. It creates the directory 
only if it does not already exist, so running it multiple 
times does not cause errors or overwrite anything.

After creating the directory and setting the correct 
permissions, I recreated the config file with nano, set 
chmod 600 on the file itself, and the ssh myec2 command 
connected successfully.

---

## Key Observations

- The SSH config file lives on your local Mac, not on the 
  EC2 instance. All setup steps happen in your Mac terminal 
  before connecting to anything remotely.
- chmod 600 on the config file is not optional! SSH 
  enforces this permission requirement as a security measure, 
  the same way it enforces chmod 400 on .pem key files.
- The HostName value must be updated every time your EC2 
  instance is stopped and restarted because AWS assigns a 
  new public IP on each start.
- You can add multiple EC2 instances to the same config 
  file using different Host nicknames: for example 
  myec2-prod, myec2-dev, myec2-project2. This scales well 
  as you add more instances across future projects.
- This lab was inspired by advice from a senior cloud 
  engineer I connected with on LinkedIn. This is a reminder that 
  networking and mentorship surface practical real-world 
  techniques that tutorials often skip.
