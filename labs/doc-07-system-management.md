# Doc 07 — System Management

## What This Documentation Covers

Knowing how to inspect the health of a Linux system 
is a baseline expectation in cloud engineering. 
Before troubleshooting a performance issue, 
right-sizing an EC2 instance, or responding to an 
alert, an engineer needs to be able to quickly read 
what the system is actually doing. This can include how much memory 
is available, how the CPU is being used, how much 
disk space remains, and where specific programs live 
on the filesystem.

This documentation covers the commands used to 
monitor system resources, inspect hardware 
information, audit command history, and locate 
binaries on a live Linux server.

---

## Commands Covered

**`history`**

Lists every command executed by the current user in 
the terminal, in the order they were run. Each entry 
is numbered, making it easy to reference or rerun 
a previous command.

```bash
history
```

In cloud environments, `history` is useful during 
incident response to reconstruct exactly what was 
run on a server before a problem occurred. It is 
also relevant in security audits where proving what 
actions a user took on a system is required. One 
important caveat: history is tied to the user session 
and can be cleared, so it should not be treated as 
a tamper-proof audit trail. For centralized and 
reliable audit logging, AWS CloudTrail is the 
appropriate solution.

---

**`free`**

Shows how much memory the system has, how much is 
currently in use, and how much is available. By 
default the output is displayed in kilobytes.

```bash
free
```

The `-m` flag converts the output to megabytes, 
which is easier to read at a glance.

```bash
free -m
```

In AWS environments, `free` is one of the first 
commands run when an application is slow or 
unresponsive. If available memory is critically low, 
the system may be swapping, which means writing 
memory contents to disk to compensate, and that 
causes significant performance degradation. This is 
a common signal that an EC2 instance needs to be 
scaled up to a larger instance type with more RAM.

---

**`/proc/meminfo`**

A file in the `/proc` directory that contains 
detailed memory information about the system. Where 
`free` gives a summary, `/proc/meminfo` gives the 
full breakdown including cache, buffers, swap usage, 
and available memory at a granular level.

```bash
cat /proc/meminfo
```

The `/proc` directory is not a real directory stored 
on disk. It is a virtual filesystem that the Linux 
kernel populates in real time with information about 
the current state of the system. Reading files from 
it is how the kernel exposes system information to 
the tools and users running on top of it.

---

**`/proc/cpuinfo`**

Contains detailed information about the CPU 
including the architecture, number of cores, model 
name, and clock speed. Each core gets its own entry 
in the output.

```bash
cat /proc/cpuinfo
```

In cloud engineering this matters when selecting 
or verifying EC2 instance types. Different instance 
families are optimized for different workloads and 
understanding what the underlying CPU looks like 
helps validate that the instance type matches the 
workload requirements.

`lscpu` is a cleaner alternative that presents the 
same information in a more readable format.

```bash
lscpu
```

---

**`uname -a`**

Displays kernel information including the kernel 
version, the hostname, the operating system, and 
the hardware architecture. The `-a` flag returns 
all available information at once.

```bash
uname -a
```

Knowing the kernel version is relevant when applying 
security patches, installing kernel-dependent 
packages, or troubleshooting compatibility issues 
between the OS and installed software.

---

**`du`**

Shows disk space usage for a specific directory and 
everything inside it. It measures how much space 
the contents of a directory are actually consuming.

```bash
du
```

The `-m` flag converts the output to megabytes.

```bash
du -m
```

The `-h` flag formats the output in human-readable 
units, automatically choosing between kilobytes, 
megabytes, and gigabytes depending on the size.

```bash
du -h /var/log
```

In AWS environments where EC2 instances use EBS 
volumes for storage, `du` is used to identify which 
directories are consuming the most space when a 
volume is filling up. Log directories under 
`/var/log` are a frequent culprit.

---

**`df`**

Shows disk space usage at the filesystem level rather 
than the directory level. Where `du` tells you how 
much space a specific directory is using, `df` tells 
you how much space is used and available across every 
mounted filesystem on the system.

```bash
df -h
```

The `-h` flag formats output in human-readable units. 
On an EC2 instance, `df -h` is the fastest way to 
see how full an EBS volume is and whether the root 
filesystem is approaching capacity. A filesystem at 
100% capacity will cause application writes to fail, 
which can bring down a running service.

---

**`top`**

Displays a real-time view of running processes, CPU 
usage, and memory consumption. It updates 
continuously and shows which processes are consuming 
the most resources at any given moment.

```bash
top
```

In cloud environments, `top` is reached for when a 
server is sluggish and the cause is not immediately 
obvious. Seeing a single process consuming 90% of 
CPU or a memory leak growing over time gives the 
information needed to decide whether to kill a 
process, restart a service, or escalate to resizing 
the instance. Press `q` to exit.

---

**`whereis`**

Returns the binary location, source code location 
if it exists, and the manual page location for a 
specified program, all at once.

```bash
whereis python3
```

`whereis` answers the question of where everything 
related to a program lives on the system. This is 
particularly useful during security audits when a 
full picture of what is installed and where it lives 
is required. If unexpected binaries show up in 
unexpected locations, that is worth investigating.

---

**`which`**

Returns the exact path of the binary that executes 
when a command is typed. It answers specifically 
what runs when a command is called, not where all 
related files live.

```bash
which python3
```

This matters in environments where multiple versions 
of the same tool are installed. Running `which 
python3` and getting back `/usr/bin/python3` confirms 
exactly which binary the system is calling. In 
security contexts, `which` is used to verify that 
the expected binary is being executed and not a 
malicious replacement placed earlier in the system 
path.

The simplest way to remember the distinction: 
`which` answers what runs when you type a command, 
and `whereis` answers where everything related to 
that program lives on the system.

---

## Real-World Scenario

A cloud engineer receives an alert that an 
application on an EC2 instance is responding slowly. 
She SSHes in and runs `top` to check whether any 
process is consuming an unusual amount of CPU or 
memory. She spots the application process using 
significantly more memory than expected. She runs 
`free -m` to confirm available memory is nearly 
exhausted and `df -h` to verify the disk is not 
also full. She checks `/proc/meminfo` for a detailed 
breakdown and uses `du -h /var/log` to confirm that 
log files are not unexpectedly consuming storage. 
With that data in hand she has enough information 
to recommend scaling the instance up to the next 
memory-optimized tier before the situation causes 
a full outage.

---

## Key Observations

* The distinction between `du` and `df` was not 
  obvious at first because they both involve disk 
  space. Once I understood that `du` measures what 
  a directory contains and `df` measures what a 
  filesystem has available, it clicked. They answer 
  different questions and you often need both to get 
  the full picture.

* I did not expect the /proc directory to work the 
  way it does. It looks like a regular directory with 
  regular files, but nothing in it is actually stored 
  on disk. The kernel writes that information in real 
  time as you read it. Coming from a background where 
  a file is just a file, that was a concept I had to 
  sit with for a minute before it made sense.

* `which` and `whereis` look like they do the same 
  thing until you run both. `which` gives you one 
  specific answer about what executes. `whereis` 
  gives you the full map of where a program lives 
  on the system. In a security audit context that 
  difference matters because you are not just asking 
  what runs, you are asking what exists and where.
