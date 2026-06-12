# Doc 05 — Linux Networking Fundamentals

## What This Documentation Covers

Networking is where Linux administration and cloud 
engineering meet directly. Every EC2 instance on AWS 
is a Linux server sitting inside a network, and the 
ability to verify connectivity, inspect what is 
listening on the system, download software, and test 
whether a deployed application is actually reachable 
are baseline expectations for anyone working in a 
cloud environment.

This documentation covers the core networking commands 
used on Linux systems in cloud environments, how they 
connect to AWS infrastructure, and where Linux-level 
networking ends and AWS-level access control begins.

---

## The Two Layers of Access Control

Before getting into individual commands, this concept 
is worth understanding first because it changes how 
you interpret everything else in this section.

When an application is running on an EC2 instance, 
there are two separate gates controlling whether 
traffic can reach it.

The first gate is the Linux system itself. A port has 
to be open and a service has to be actively listening 
on that port before anything can connect to it.

The second gate is the AWS security group attached to 
the instance. Even if a port is open and a service is 
listening on the Linux side, the security group has to 
explicitly allow inbound traffic on that port or the 
connection will be blocked before it ever reaches the 
server.

Both gates have to be open. A misconfigured security 
group is one of the most common reasons a deployed 
application is unreachable even when everything on the 
Linux side looks correct. Understanding that these are 
two separate layers prevents a lot of unnecessary 
troubleshooting time spent looking in the wrong place.

---

## Commands Covered

**`hostname`**

Reads the name assigned to the server. Running it 
without any arguments simply returns the current 
hostname. The hostname is stored permanently in 
`/etc/hostname` and any change made to that file 
requires a system restart to take effect.

```bash
hostname
```

To change the hostname temporarily for the current 
session without editing any files:

```bash
hostname newname
```

To change it permanently without requiring a restart, 
`hostnamectl` is the modern standard on current Linux 
systems including Amazon Linux:

```bash
hostnamectl set-hostname newname
```

One important caution in production environments: 
changing a hostname on a server that is already 
registered in DNS can break external network 
communication. This is not a command to run casually 
on a live system.

---

**`ping`**

Sends a signal to a target IP address or domain name 
and measures whether a response comes back. It is the 
fastest way to verify basic network connectivity 
between two systems.

```bash
ping google.com
ping 10.20.30.40
```

If responses come back, the target is reachable. If 
packets are lost or there is no response, the target 
is either down, unreachable from the current network, 
or blocked by a firewall. In AWS environments, a 
failed ping does not always mean the server is down — 
security groups can be configured to block ICMP 
traffic, which is what ping uses, while still allowing 
other types of traffic through.

---

**`wget`**

Downloads files and packages from the internet 
directly onto the Linux system. Commonly used to pull 
down application packages, scripts, or configuration 
files onto an EC2 instance without going through a 
package manager.

```bash
wget https://example.com/file.tar.gz
```

The file downloads into the current working directory 
by default.

---

**`curl`**

Makes HTTP requests from the terminal and returns the 
response. Where `wget` is for downloading files, 
`curl` is for interacting with web servers and APIs 
directly from the command line.

```bash
curl https://example.com
```

In cloud engineering, `curl` is most commonly used 
to verify that a deployed application is actually 
responding after a deployment. Rather than opening a 
browser, an engineer can test an endpoint directly 
from the server or from another instance in the same 
network.

The `-I` flag returns only the response headers, which 
is useful for quickly checking the HTTP status code 
of a running service without pulling the full response 
body.

```bash
curl -I https://example.com
```

The `-L` flag tells curl to follow redirects 
automatically, which is useful when a URL redirects 
to another location.

```bash
curl -L https://example.com
```

---

**`ifconfig`**

Displays the network interfaces on the system and 
their associated IP addresses. On an EC2 instance 
this shows the private IP address assigned to the 
instance within the VPC.

```bash
ifconfig
```

`ifconfig` is an older command that is being phased 
out on modern Linux systems. The current replacement 
is `ip addr`, which provides the same information in 
a more consistent format.

```bash
ip addr
```

In AWS, EC2 instances have both a private IP address 
assigned within the VPC and an optional public IP 
address for internet-facing access. The address 
visible through `ifconfig` or `ip addr` will be the 
private IP. The public IP is managed by AWS and 
visible through the console or the instance metadata 
service.

---

**`netstat`**

Displays active network connections and the ports 
currently in use on the system. The most useful flag 
combination in a cloud engineering context shows all 
listening ports, the protocol, and the process using 
each port.

```bash
netstat -tulpn
```

Breaking down those flags: `-t` filters for TCP 
connections, `-u` includes UDP, `-l` shows only 
listening ports, `-p` shows the process name and ID 
using each port, and `-n` displays addresses and port 
numbers numerically rather than resolving them to 
names.

`netstat` is being replaced by `ss` on modern Linux 
systems. Both commands serve the same purpose but `ss` 
is faster and better supported going forward.

```bash
ss -tulpn
```

In a cloud environment, running either of these 
commands on an EC2 instance before opening a port in 
a security group confirms that a service is actually 
listening before any inbound rules are changed. It 
also helps identify unexpected processes listening on 
ports they should not be using, which is relevant 
during security audits.

---

**`telnet`**

Tests whether a specific port is open and accepting 
connections on a target system. Despite being an older 
protocol, it remains useful as a diagnostic tool for 
port connectivity checks.

```bash
telnet hostname 80
telnet localhost 22
```

If the connection succeeds, the port is open and 
something is listening on it. If the connection is 
refused, nothing is running on that port. This is a 
quick way to verify connectivity at the port level 
rather than just at the network level the way `ping` 
does.

---

**`nslookup`**

Queries DNS to resolve a domain name to its IP 
address, or reverse-resolves an IP address to a 
domain name. In cloud environments, DNS issues are 
a frequent source of connectivity problems that look 
like network failures on the surface.

```bash
nslookup google.com
nslookup 8.8.8.8
```

On AWS, Route 53 handles DNS for cloud-hosted 
applications. When a newly deployed application is 
not resolving correctly, `nslookup` is the first tool 
reached for to determine whether the problem is in 
DNS configuration or somewhere else in the stack.

---

## Real-World Scenario

A cloud engineer is asked to verify that a web 
application deployed on an EC2 instance is accessible 
from outside the VPC. She starts by SSHing into the 
instance and running `ss -tulpn` to confirm the 
application is listening on port 80. It is. She then 
runs `curl -I http://localhost` to verify the service 
is responding with a 200 status code locally. It is. 
She checks the AWS console and finds the security 
group attached to the instance has no inbound rule 
for port 80. The application was running correctly 
the entire time. The security group was the second 
gate that had never been opened. She adds the rule 
and the application becomes immediately accessible.

---

## Troubleshooting

**`ping` returns no response but the server is running**
In AWS, security groups can block ICMP traffic, which 
is what ping uses. A server that does not respond to 
ping is not necessarily down. Test connectivity using 
`telnet` or `curl` on a specific port instead to get 
a more accurate picture.

**Application is listening but still unreachable**
If `ss -tulpn` shows a service listening on the 
correct port but the application cannot be reached 
from outside, the issue is almost always the AWS 
security group. Confirm that an inbound rule exists 
for the correct port and protocol before investigating 
anything on the Linux side.

**`ifconfig` command not found**
On newer Linux distributions `ifconfig` is not 
installed by default. Use `ip addr` instead, which 
is the current standard and available on all modern 
systems.

---

## Key Observations

* The most important thing I took from this section 
  was not any individual command. It was understanding 
  that Linux networking and AWS networking are two 
  separate layers. A port can be open on the server 
  and a service can be running, and the application 
  can still be completely unreachable if the security 
  group has not been configured. Knowing to check both 
  layers before troubleshooting saves a significant 
  amount of time.

* Learning that `netstat` and `ifconfig` are both 
  being phased out in favor of `ss` and `ip addr` was 
  a useful reminder that cloud engineering requires 
  staying current. Knowing the legacy commands matters 
  because they still appear in older documentation and 
  tutorials, but defaulting to the modern replacements 
  is the right habit to build now.
