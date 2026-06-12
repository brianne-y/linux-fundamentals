<h1 align="center">Linux Fundamentals</h1>

<p align="center">
  <img src="images/tux.png" alt="Linux Fundamentals" width="600"/>
</p>

Hands-on lab documentation covering Linux fundamentals from 
the ground up. Every concept practiced on a live Amazon Linux 
2023 EC2 instance, documented publicly, and connected to how 
Linux is actually used in production cloud environments.

Linux is the operating system running underneath nearly every 
cloud server, container, and production workload in existence. 
Engineers who work in cloud infrastructure do not just run 
commands. They understand why the Linux filesystem is 
structured the way it is, what permissions actually protect, 
and how the terminal connects to every layer of the cloud 
engineering stack.

---

## What's Covered

- SSH configuration and remote server access
- Filesystem structure and hierarchy
- File viewing, creation, and management
- Text searching and command chaining
- File permissions and access control
- User and group administration
- Linux networking fundamentals
- Service management with systemctl
- System management 
- Bash scripting and task automation
- Log analysis and system monitoring


---
| Title | Topic | Status |
|---|---|---|
| [Lab 01 — SSH into EC2 from Mac Terminal](labs/lab-01-ssh-into-ec2.md) | SSH, key pair authentication, AWS CLI verification | ✅ Complete |
| [Lab 02 — SSH Config File: Connecting to EC2 Made Easy](labs/lab-02-ssh-config-file.md) | SSH config file setup, hostname aliasing, permission hardening | ✅ Complete |
| [Doc 03 — File System Navigation](labs/doc-03-file-system-navigation.md) | Filesystem hierarchy, pwd, ls, cd, mkdir, rm, cp, mv, find, cat, touch, nano, grep, pipe operator | ✅ Complete |
| [Doc 04 — File Permissions, Ownership, and User Management](labs/doc-04-file-permissions-ownership-user-management.md) | chmod, chown, useradd, usermod, passwd, id, groups | ✅ Complete |
| [Doc 05 — Linux Networking Fundamentals](labs/doc-05-linux-networking-fundamentals.md) | hostname, ping, wget, ifconfig, ip addr, curl, netstat, ss, telnet, nslookup | ✅ Complete |
| [Doc 06 — Service and Package Management](labs/doc-06-service-and-package-management.md) | systemctl, yum, reload, restart, enable, disable, journalctl | ✅ Complete |
| [Doc 07 — System Management](labs/doc-07-system-management.md) | history, free, uname, du, whereis, which, /proc/meminfo, /proc/cpuinfo | 🔨 In Progress |
| [Doc 08 — Nginx Web Server on EC2](labs/doc-08-nginx-web-server-on-ec2.md) | Package installation, service configuration, live deployment | 📋 Planned |
| [Doc 09 — Log Analysis](labs/doc-09-log-analysis.md) | tail, grep, awk, CloudWatch log basics | 📋 Planned |
| [Doc 10 — Bash Automation Script](labs/doc-10-bash-automation-script.md) | Writing and executing shell scripts on EC2 | 📋 Planned |


## Environment

Amazon Linux 2023 on AWS EC2 (t2.micro)  
Access method: SSH via local Mac terminal using key pair 
authentication and SSH config file aliasing
