# Linux Fundamentals

Hands-on lab documentation and command reference covering 
Linux administration in cloud environments. All labs are 
executed on Amazon Linux 2023 EC2 instances accessed via 
SSH from a local Mac terminal: reflecting real-world cloud 
infrastructure management workflows.

Currently 30 days into a dedicated Linux fundamentals run, 
documenting every lab, command, and troubleshooting 
experience in public as part of an active cloud engineering 
career transition.

---

## Coverage

- SSH configuration and remote server access
- File system navigation and management
- File permissions, ownership, and access control
- User and group administration
- Service management with systemctl
- Network diagnostics and connectivity troubleshooting
- Bash scripting for task automation
- Log analysis and system monitoring

---

## Labs

| Lab | Topic | Status |
|---|---|---|
| [Lab 01 — SSH into EC2 from Mac Terminal](labs/lab-01-ssh-into-ec2.md) | SSH, key pair authentication, AWS CLI verification | ✅ Complete |
| [Lab 02 — SSH Config File: Connecting to EC2 Made Easy](labs/lab-02-ssh-config-file.md) | SSH config file setup, hostname aliasing, permission hardening | ✅ Complete |
| Lab 03 — File System Navigation | cd, ls, pwd, mkdir, rm, cp, mv | 📋 Planned |
| Lab 04 — File Permissions and Ownership | chmod, chown, user and group management | 📋 Planned |
| Lab 05 — Service Management | systemctl, starting and stopping services | 📋 Planned |
| Lab 06 — Nginx Web Server on EC2 | Package installation, service configuration, live deployment | 📋 Planned |
| Lab 07 — Log Analysis | tail, grep, awk, CloudWatch log basics | 📋 Planned |
| Lab 08 — Bash Automation Script | Writing and executing shell scripts on EC2 | 📋 Planned |

---

## Command Reference

| Category | File |
|---|---|
| Navigation | [navigation.md](command-reference/navigation.md) |
| File Management | [file-management.md](command-reference/file-management.md) |
| Permissions | [permissions.md](command-reference/permissions.md) |
| Networking | [networking.md](command-reference/networking.md) |
| Services | [services.md](command-reference/services.md) |
| Scripting | [scripting.md](command-reference/scripting.md) |

---

## Environment

Amazon Linux 2023 on AWS EC2 (t2.micro)
Access method: SSH via local Mac terminal using key pair 
authentication and SSH config file aliasing
