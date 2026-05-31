# Doc 04 — File Permissions, Ownership, and User Management

## What This Documentation Covers

File permissions and user management are at the core of how Linux controls who can access what. Every file has an owner, a group, and a set of rules that define what each party is allowed to do with it. Learning how to read, set, and modify those rules, and understanding how user accounts are created and managed, is what makes it possible to work safely and responsibly on a live server.

This documentation covers the permission string, `chmod`, `chown`, `useradd`, `usermod`, `passwd`, `groups`, and `id`.

---

## Understanding the Linux Permission String

Every file and directory on a Linux system carries a permission string visible when running `ls -l`. The string is broken into four parts: the file type indicator, the owner's permissions, the group's permissions, and the permissions for everyone else.

The first character identifies the file type. A dash means it is a regular file. The letter `d` means it is a directory.

The remaining nine characters are split into three groups of three. Each group represents read (`r`), write (`w`), and execute (`x`) for the owner, the group, and all other users respectively. A dash in any position means that permission is not granted.

Permissions behave differently depending on whether they are applied to a file or a directory.

| Permission | On a File | On a Directory |
|---|---|---|
| `r` (read) | Open and view the file | List the contents with `ls` |
| `w` (write) | Edit, append, or delete the file | Add, delete, or rename contents |
| `x` (execute) | Run the file as a command or script | Enter the directory using `cd` |

---

## Commands Covered

**`chmod`**

`chmod` (change mode) sets who can read, write, or execute a file or directory. There are two ways to use it.

The symbolic method uses letters and symbols. `u` targets the owner, `g` targets the group, and `o` targets everyone else. The `+` symbol adds a permission, `-` removes one, and `=` replaces the existing permissions entirely.

```bash
chmod u+x filename
chmod u=rwx,g=r,o=r filename
```

The absolute method, also called numeric, uses numbers instead. Read is 4, write is 2, and execute is 1. The values for each target are added together and written side by side as a three-digit number.

```bash
chmod 755 filename
chmod 644 filename
```

The three values worth memorizing:

| Value | Permissions | Common Use |
|---|---|---|
| `777` | Everyone gets full access | Rarely appropriate |
| `755` | Owner has full access, everyone else can read and execute | Standard for directories |
| `644` | Owner can read and write, everyone else can read only | Standard for files |

`chmod` can only be performed by the root user or the file's owner.

---

**`chown`**

`chown` (change owner) transfers ownership of a file or directory. Only the root user can run it.

To change only the user owner:

```bash
chown username filename
```

To change both the user owner and the group at the same time, separate them with a colon:

```bash
chown username:groupname filename
```

---

**`useradd`**

`useradd` creates a new user account and sets up its default environment. In cloud engineering, this command is rarely typed manually. It is more commonly embedded inside automation scripts, AWS cloud-init configurations, and Dockerfiles to provision accounts programmatically.

```bash
useradd [options] username
```

The `-m` flag creates a home directory for the new user under `/home`.

```bash
useradd -m username
```

The `-s` flag defines the login shell, which is the interface assigned to the user when they authenticate. Standard interactive shells like `/bin/bash` are used for human accounts. Service accounts running background applications are assigned `/usr/sbin/nologin`, which blocks interactive access entirely.

```bash
useradd -s /bin/bash username
useradd -s /usr/sbin/nologin username
```

The `-u` flag forces a specific numeric user ID instead of letting the system assign one automatically. This matters when managing shared storage volumes like AWS EFS or Docker containers across multiple servers — matching UIDs ensures the filesystem recognizes the same identity across environments.

```bash
useradd -u 1050 username
```

The `-g` flag assigns the user's primary group, which automatically owns every file the user creates.

```bash
useradd -g groupname username
```

The `-G` flag assigns the user to one or more supplementary groups for additional access. Multiple groups are separated by commas with no spaces.

```bash
useradd -G groupname,secondgroupname username
```

---

**`usermod`**

`usermod` modifies an existing user account. It always requires at least one flag. Running it without one produces an error because the system has nothing to act on.

The `-l` flag renames the account's login name.

```bash
usermod -l newusername oldusername
```

The `-aG` flag adds the user to a supplementary group without touching their existing memberships. The `-a` is not optional — leaving it out silently removes all existing secondary group assignments and replaces them with only the one specified.

```bash
usermod -aG groupname username
```

The `-d` flag updates the home directory path assigned to the user.

```bash
usermod -d /new/path/username username
```

The `-L` flag locks the account by blocking the password, immediately cutting off login access without deleting the account.

```bash
usermod -L username
```

The `-U` flag reverses the lock and restores login access.

```bash
usermod -U username
```

The `-s` flag changes the active shell. A shell is the interface program between the user and the operating system — it handles input, tab completions, and command shortcuts. Swapping it out can either upgrade a user's terminal experience or lock the account down completely.

```bash
usermod -s /path/to/shell-program username
```

---

**`passwd`**

`passwd` sets, updates, or restricts the password for an account. In its basic form it prompts for a new password.

```bash
passwd username
```

The `-d` flag removes the password requirement entirely, forcing the account to use alternative authentication like SSH key pairs.

```bash
passwd -d username
```

The `-l` flag locks the account by inserting an exclamation point at the beginning of the encrypted password string inside `/etc/shadow`, which is the secure file where Linux stores encrypted credential data. The password is not deleted — the account is just made inaccessible, which keeps configuration and audit history intact.

```bash
passwd -l username
```

The `-u` flag removes the exclamation point and restores access.

```bash
passwd -u username
```

---

**`groups`**

`groups` returns the names of every group the specified user belongs to. It is read-only and changes nothing.

```bash
groups username
```

---

**`id`**

`id` returns the numeric user ID, primary group ID, and full list of group memberships for an account. Useful for verifying that a user has the access they are supposed to have.

```bash
id username
```

---

## Real-World Scenario

A new engineer joins a team and needs read access to application logs on an EC2 instance but should not be able to modify files or escalate privileges. The administrator creates the account with `useradd -m -s /bin/bash`, adds it to the logs group with `-G`, and sets the log files to `chmod 640` so the group can read but not write. She confirms everything looks right with `id` and `groups` before closing the ticket. When the engineer moves to a different project, instead of deleting the account and losing the audit trail, she runs `passwd -l` to lock it immediately while keeping the account configuration in place.

---

## Troubleshooting

**`usermod -G` removed existing group memberships**
Running `usermod -G groupname username` without `-a` replaces all existing supplementary groups with only the one listed. The correct command is `usermod -aG groupname username`. The `-a` flag must always be present when the goal is to add rather than replace.

**`useradd` skipped creating the home directory**
On enterprise Linux servers, `useradd username` without `-m` often does not create the home directory at all. The fix is to always include `-m` for accounts that will be used interactively.

---

## Key Observations

* I came into this session thinking the symbolic method for `chmod` would be straightforward. It was not. Tracking the target, the operator, and the permission letter at the same time felt like too many moving pieces. The absolute method made more sense to me faster because the logic is consistent and the math is simple.

* I assumed Linux automatically creates a home directory every time a new user account is added, the same way macOS does. That assumption was wrong. On enterprise Linux servers, running `useradd` without `-m` skips the home directory entirely. That was a real correction to how I was thinking about account creation.

* While studying the `-u` flag, I stopped and asked myself why UID matching even matters when you could just use `chmod` to set permissions. I worked through it and realized that `chmod` works cleanly on a single machine, but in a distributed environment with multiple servers and containers, relying on it creates security gaps, automation overhead, and isolation problems. UIDs solve the problem at the identity level before permissions even enter the conversation.

* The `-aG` flag caught me off guard. I did not expect that forgetting a single letter could silently remove all of a user's existing group memberships. No error, no warning, just gone. That is the kind of behavior that causes real access issues in production and it is the detail I will not forget.
