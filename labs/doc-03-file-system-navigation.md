# Doc 03 — File System Navigation & File Management

## What This Documentation Covers

Understanding how Linux organizes its filesystem, how to move through it, create and delete files, read content, and search for what you need is the foundation every other cloud engineering skill is built on. Every time an engineer connects to an EC2 instance, they are dropped into this filesystem with nothing but a terminal. There is no graphical interface, no file explorer, no drag and drop. Navigation, file management, and text searching all happen through commands.

In AWS environments specifically, this matters from day one. Locating configuration files before a deployment, reviewing logs during an incident, creating scripts that run on a schedule, verifying that a service is reading from the right directory — none of that is possible without a solid command of the filesystem. For engineers working in federal contracting, healthcare IT, or cybersecurity, where systems are audited and access is controlled, knowing exactly where things live and how to find them quickly is not a nice-to-have. It is an essential part of the job.

---

## The Linux Filesystem Hierarchy

Linux organizes everything under a single top-level directory called the root directory, represented by a forward slash (`/`). Every file, folder, and device on the system lives somewhere beneath that point. Understanding what each major directory contains is what allows an engineer to navigate an unfamiliar server with confidence.

**`/`** is the root directory and the starting point of the entire filesystem. Every path on the system begins here.

**`/root`** is the home directory for the root user, which is the system's superuser, meaning an account with unrestricted access to everything on the system. It is separate from `/home` because it needs to be accessible even in minimal or recovery boot states.

**`/home`** is the home directory for all standard users. When a regular user logs in, they land in their personal subdirectory here. On EC2 instances, the default user such as `ec2-user` on Amazon Linux has a home directory under `/home`.

**`/etc`** stores configuration files for the system and installed software. When engineers configure services like Nginx or SSH, those files live here. It is frequently referenced during deployments and security hardening.

**`/tmp`** holds temporary files that are wiped on every reboot. It is useful for short-lived files, but anything that needs to persist across restarts should never be stored here.

**`/usr`** stands for Unix Shareable Resources and is where software installed on the system lives by default. It is the largest directory on most systems. When a package is installed via a package manager like `yum`, its files typically end up here.

**`/bin`** contains core commands available to all users, specifically the binary executable files the system needs to function. Commands like `ls`, `cat`, `cp`, and `mv` live here.

**`/sbin`** is similar to `/bin` but the commands here are reserved for system administration and the root user. Commands used to manage system processes or configure networking typically live here.

**`/var`** stores variable data, meaning content that changes constantly as the system runs. Log files, mail spools, and database files all live here. In AWS environments, logs from `/var/log` are frequently streamed to CloudWatch for centralized monitoring and retention.

---

## Commands Covered

**`date`**
Shows the current date and time on the system. Used to verify the system clock, which matters for log correlation, certificate validity, and scheduled jobs on EC2 instances.

**`cal`**
Displays a calendar for the current month in the terminal. Useful for quick date references during incident documentation or scheduling.

**`uptime`**
Shows how long the system has been running. It is one of the first things checked during incident response because an unexpected reboot on a production instance could indicate a kernel crash, a failed deployment, or a scheduled maintenance event.

**`whoami`**
Returns the username of the currently logged-in user. It is a basic safety check before running commands, confirming whether a session is running as `ec2-user` or `root` to prevent accidental privilege escalation.

**`uname`**
Displays system information including the kernel version and architecture. This is relevant when installing packages or troubleshooting compatibility issues on an EC2 instance.

**`man`**
Displays the full manual page, which is the built-in documentation, for any command that follows it. In environments where internet access is restricted, which is common in federal and secure healthcare networks, `man` is the primary reference for command documentation.

**`ls`**
Lists the contents of a directory. On its own it gives a clean minimal view, but its flags significantly expand what it shows.

The `-l` flag switches to long format, showing file permissions, ownership, size, and the date last modified. The `-a` flag shows all files including hidden ones. Hidden files in Linux start with a dot, so configuration files like `.bashrc` and `.ssh/config` are invisible without this flag. The `-t` flag sorts results by the time files were last modified, with the most recent first, which is useful for identifying recently updated files on a server. The `-r` flag reverses the sort order.

Flags can be combined into a single flag. Writing `ls -ltr` produces the same output as `ls -l -t -r`, giving a long-format listing sorted oldest to newest. In long-format output, directories are identified by the letter `d` at the start of the permissions string, while files show a dash in that position.

**`cat`**
Outputs the full contents of a file to the terminal at once. It is best for short files and not suited for large files since it dumps everything without any ability to scroll or navigate.

**`less`**
Displays file content one screen at a time with the ability to scroll forward and backward. It is the better choice for large files, particularly configuration files and log output.

**`more`**
Similar to `less` but only allows forward navigation. It is an older command that has largely been replaced by `less` in modern usage.

**`head`**
Outputs the first 10 lines of a file by default. It is used to quickly check the beginning of a log file or verify the header of a configuration file.

**`tail`**
Outputs the last 10 lines of a file by default. In production environments, `tail` is one of the most frequently used commands because it shows what just happened on a running system without loading the entire file.

**`touch`**
Creates an empty file. If the file already exists, `touch` updates its timestamp without changing the content. It is the fastest way to create a placeholder file or verify that a path is writable.

**`cat >`**
The `cat` command paired with the redirect operator `>` creates a file and allows content to be written into it directly from the terminal. The `>` operator overwrites any existing content. Using `>>` instead appends to the end of an existing file without touching what is already there. Using `>` when `>>` was intended will silently destroy existing file content, which makes this one of the more consequential distinctions to get right in production.

**`nano`**
A terminal-based text editor, meaning software that allows text to be written and edited within the terminal window. It is more beginner-friendly than `vi` because its commands are displayed at the bottom of the screen. The workflow is to open or create a file with `nano [filename]`, write the content, save with `Ctrl+O`, confirm with Enter, and then exit with `Ctrl+X`.

**`vi`**
A more complex terminal-based text editor that operates in separate modes for navigation and text insertion. It is available on virtually every Unix-based system, which makes it useful when other editors are not installed.

**`mkdir`**
Creates a new directory. Multiple directories can be created in a single command by listing names separated by spaces, which is useful when setting up an entire directory structure during application provisioning.

**`cd`**
Changes the current working directory, which is the location in the filesystem where the terminal session is currently positioned. It is used to navigate into newly created directories or move between locations in the filesystem.

**`rm`**
Removes files and can remove multiple files in a single command. The `-f` flag, which stands for force, executes the removal without asking for confirmation. There is no trash folder or undo on a Linux server, so this flag should be used carefully. The `-r` flag, which stands for recursive, removes a directory and everything inside it. It is required when a directory has content since `rm` alone cannot delete a non-empty directory.

**`rmdir`**
Removes an empty directory and refuses to delete a directory that still has content inside it. It is a safer option when working with directories that should already be empty.

**`cp`**
Copies a file from one location to another. The source file stays in its original location. The syntax puts the source file first, followed by the destination.

**`mv`**
Moves a file from one location to another, removing it from the original location. It is also used to rename files because moving a file to the same directory under a different name is how renaming works in Linux.

**`find`**
Searches the filesystem for files or directories matching specified criteria, starting from a given path and searching every subdirectory from that point. The `-name` flag searches by filename. The `-user` flag finds files owned by a specific user. The `-group` flag finds files belonging to a specific group.

**`diff`**
Compares two files and shows the differences between them. If the files are identical, no output appears. It is commonly used in production to compare a current configuration file against a known-good backup.

**`file`**
Identifies what type of content a file contains regardless of its extension. Linux does not rely on file extensions to determine file types, which makes this command useful when working with files on unfamiliar systems.

**`grep`**
Stands for Global Regular Expression Print. It searches for a word or pattern within a file and returns every line that contains a match. The search is case-sensitive by default. The `-i` flag makes the search case-insensitive. The caret symbol `^` used inside a search pattern restricts matches to lines where the pattern appears at the very beginning of the line.

**Pipe (`|`)**
Connects two commands by sending the output of the first directly into the input of the second. This allows multiple operations to be combined without creating intermediate files. Running `ls | grep "^d"` for example sends the directory listing from `ls` into `grep`, which then returns only the entries starting with `d`. Multiple commands can be chained together in sequence.

**`sudo su`**
Switches the terminal session to the root user. The `sudo` command executes a single command with elevated privileges, and `su` opens a full session as a specified user. Combined, `sudo su` opens a root session on systems where the current user has the necessary permissions. Typing `exit` ends the root session and returns to the standard user.

---

## Real-World Scenario

A cloud engineer supporting a federal agency's web application is asked to investigate why an application behaved unexpectedly overnight. She SSHes into the EC2 instance, runs `uptime` to check whether the server restarted, and uses `whoami` to confirm her permission level before touching anything. She navigates to `/var/log` using `cd`, uses `tail` to check the most recent log entries, then pipes the output through `grep` to filter for error messages. She uses `diff` to compare the current configuration file in `/etc` against the backup copy to check whether anything changed. She finds the issue, copies the backup into place using `cp`, and confirms the fix. The entire investigation happens in the terminal without downloading a single file.

---

## Troubleshooting

**Cannot delete a directory with `rmdir`**
The `rmdir` command only removes empty directories. If the directory has content, the options are to remove the contents first or use `rm -r` to delete the directory and everything inside it recursively.

**`cat >` overwrites an existing file**
Using `cat > filename` on a file that already exists will silently replace all of its content. If the intent was to add to the file rather than replace it, `cat >> filename` is the correct form. The terminal gives no warning before overwriting.

---

## Key Observations

I did not expect the filesystem to feel as logical as it does. Once I understood that `/bin` holds commands for everyone and `/sbin` is for root-level administration, I stopped guessing and started seeing the intention behind the structure. The `>` versus `>>` distinction is one that genuinely matters in production because accidentally overwriting a configuration file is the kind of mistake that takes down a running service, and that consequence made it stick immediately. Understanding that `mv` is how you rename files also clicked in a way that made the whole system feel more coherent because there is no separate rename command, moving and renaming are the same operation, and once you see that it just makes sense.
