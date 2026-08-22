# 🔥 Firewalld Management with Ansible

This guide explains how to manage **firewalld** on Linux systems using **Ansible**.

> ⚠️ **Note:** Some commands from the original notes contained spelling or syntax mistakes. The commands below are corrected.

---

## 📌 Table of Contents

* [1. Check Firewalld Installation](#1-check-firewalld-installation)
* [2. Start Firewalld](#2-start-firewalld)
* [3. Check Firewalld Status](#3-check-firewalld-status)
* [4. Check Firewalld Version](#4-check-firewalld-version)
* [5. Firewalld Zones](#5-firewalld-zones)
* [6. Services and Ports](#6-services-and-ports)
* [7. Open a Port](#7-open-a-port)
* [8. Open a Port Range](#8-open-a-port-range)
* [9. Remove a Port Range](#9-remove-a-port-range)
* [10. Working with Zones](#10-working-with-zones)
* [11. Network Interfaces](#11-network-interfaces)
* [12. Rich Rules](#12-rich-rules)
* [13. Runtime vs Permanent](#13-runtime-vs-permanent)
* [14. Quick Command Reference](#14-quick-command-reference)
* [15. Common Corrections](#15-common-corrections)
* [16. Basic Practice Lab](#16-basic-practice-lab)
* [17. Important Safety Note](#17-important-safety-note)

---

# 1. Check Firewalld Installation

```bash
ansible all -m shell -a "rpm -qa | grep firewalld"
```

### Explanation

* `ansible all` → Runs the command on all hosts in the Ansible inventory.
* `-m shell` → Uses the Ansible shell module.
* `-a` → Specifies the command to execute.
* `rpm -qa` → Lists installed RPM packages.
* `grep firewalld` → Searches for packages containing `firewalld`.

If firewalld is installed, you should see something similar to:

```text
firewalld-1.x.x
```

---

# 2. Start Firewalld

```bash
ansible all -m shell -a "systemctl start firewalld"
```

This starts the `firewalld` service.

### Recommended Ansible method

Instead of using `shell`, you can use the Ansible `service` module:

```bash
ansible all -b -m service -a "name=firewalld state=started"
```

* `-b` → Enables privilege escalation (`sudo`).
* `state=started` → Ensures the service is running.

---

# 3. Check Firewalld Status

```bash
ansible all -m shell -a "systemctl status firewalld --no-pager"
```

This checks whether firewalld is running.

Look for:

```text
Active: active (running)
```

### `--no-pager`

Prevents `systemctl` from opening the output in a pager.

---

# 4. Check Firewalld Version

```bash
ansible all -m shell -a "firewall-cmd --version"
```

Displays the installed firewalld version.

### ❌ Incorrect

```bash
firewall-cmd --virsion
```

### ✅ Correct

```bash
firewall-cmd --version
```

---

# 5. Firewalld Zones

Firewalld uses **zones** to define different levels of trust for network connections.

Common zones include:

```text
drop
block
public
external
dmz
work
home
internal
trusted
```

---

## 5.1 Check Default Zone

```bash
ansible all -m shell -a "firewall-cmd --get-default-zone"
```

Shows the default firewall zone.

Example:

```text
public
```

---

## 5.2 Check Available Zones

```bash
ansible all -m shell -a "firewall-cmd --get-zones"
```

Lists all available firewall zones.

### ❌ Incorrect

```bash
firewall-cmd --get-zone
```

### ✅ Correct

```bash
firewall-cmd --get-zones
```

---

## 5.3 Check Active Zones

```bash
ansible all -m shell -a "firewall-cmd --get-active-zones"
```

Displays currently active zones and their associated network interfaces.

Example:

```text
public
  interfaces: ens160
```

---

## 5.4 Display Complete Zone Configuration

```bash
ansible all -m shell -a "firewall-cmd --list-all"
```

Displays the configuration of the default zone.

It can show:

* Interfaces
* Sources
* Services
* Ports
* Protocols
* Masquerading
* Forward ports
* Rich rules

---

# 6. Services and Ports

## 6.1 List Allowed Services

```bash
ansible all -m shell -a "firewall-cmd --list-services"
```

Shows services currently allowed in the default zone.

Example:

```text
cockpit dhcpv6-client ssh
```

---

## 6.2 List Allowed Ports

```bash
ansible all -m shell -a "firewall-cmd --list-ports"
```

Shows manually opened ports.

Example:

```text
80/tcp 8080/tcp
```

---

# 7. Open a Port

## 7.1 Permanently Open HTTP Port 80

```bash
ansible all -m shell -a "firewall-cmd --add-port=80/tcp --permanent"
```

This permanently opens TCP port `80`.

### Explanation

```text
--add-port=80/tcp
```

Opens TCP port 80.

```text
--permanent
```

Saves the rule to the permanent firewall configuration.

---

## 7.2 Reload Firewalld

After adding a permanent rule:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

This reloads the firewall configuration.

---

## 7.3 Verify Port 80

```bash
ansible all -m shell -a "firewall-cmd --list-ports"
```

Expected output:

```text
80/tcp
```

---

# 8. Open a Port Range

To open ports `5000` through `5010`:

```bash
ansible all -m shell -a "firewall-cmd --add-port=5000-5010/tcp --permanent"
```

This opens:

```text
5000
5001
5002
...
5010
```

for TCP traffic.

---

## Reload Firewalld

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

---

## Verify

```bash
ansible all -m shell -a "firewall-cmd --list-ports"
```

Expected:

```text
5000-5010/tcp
```

---

# 9. Remove a Port Range

Remove ports `5000-5010`:

```bash
ansible all -m shell -a "firewall-cmd --remove-port=5000-5010/tcp --permanent"
```

Reload:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

Verify:

```bash
ansible all -m shell -a "firewall-cmd --list-ports"
```

---

# 10. Working with Zones

## 10.1 Display Public Zone

```bash
ansible all -m shell -a "firewall-cmd --zone=public --list-all"
```

Displays all settings for the `public` zone.

### ❌ Incorrect

```bash
firewall-cmd --zone=public--list-all
```

### ✅ Correct

```bash
firewall-cmd --zone=public --list-all
```

There must be a space between:

```text
public
```

and:

```text
--list-all
```

---

# 10.2 Allow HTTP Service in Public Zone

```bash
ansible all -m shell -a "firewall-cmd --zone=public --permanent --add-service=http"
```

This permanently allows the HTTP service in the `public` zone.

Reload:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

Verify:

```bash
ansible all -m shell -a "firewall-cmd --zone=public --list-services"
```

You should see:

```text
http
```

---

# 11. Network Interfaces

## 11.1 Check IP Addresses

```bash
ansible all -b -m shell -a "ip -br addr"
```

Displays network interfaces and their IP addresses in a short format.

Example:

```text
lo       UNKNOWN 127.0.0.1/8
ens160   UP      192.168.10.20/24
```

---

## 11.2 Add Interface to Public Zone

```bash
ansible all -b -m shell -a "firewall-cmd --permanent --zone=public --add-interface=ens160"
```

This permanently associates `ens160` with the `public` zone.

Reload:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

Verify:

```bash
ansible all -m shell -a "firewall-cmd --zone=public --list-all"
```

Look for:

```text
interfaces: ens160
```

---

# 11.3 Change Default Zone

```bash
ansible all -b -m shell -a "firewall-cmd --permanent --set-default-zone=private"
```

Changes the default zone to:

```text
private
```

Reload:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

Verify:

```bash
ansible all -m shell -a "firewall-cmd --get-default-zone"
```

Expected:

```text
private
```

### ⚠️ Warning

Changing the default zone can change firewall behavior for network interfaces.

Be careful when doing this on a remote server because you could lose network access.

---

# 11.4 Check Network Configuration with ifconfig

```bash
ansible all -m shell -a "ifconfig"
```

Displays network interface information.

On modern Linux systems, `ip` is usually preferred:

```bash
ansible all -m shell -a "ip addr"
```

---

# 12. Rich Rules

**Rich rules** allow more advanced firewall configurations.

They can be used to:

* Allow traffic from a specific IP
* Reject traffic from a specific IP
* Allow a service for a specific source
* Apply more detailed firewall conditions

---

# 12.1 Reject Traffic from a Specific IP

Example IP:

```text
192.168.10.10
```

Command:

```bash
ansible all -m shell -a "firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.10.10 reject'"
```

This permanently adds a rule that rejects traffic from:

```text
192.168.10.10
```

### Explanation

```text
family=ipv4
```

Applies the rule to IPv4 traffic.

```text
source address=192.168.10.10
```

Specifies the source IP.

```text
reject
```

Rejects the traffic.

---

## 12.2 Reload Firewall

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

---

## 12.3 List Rich Rules

```bash
ansible all -m shell -a "firewall-cmd --list-rich-rules"
```

Displays the configured rich rules.

---

# 12.4 Remove the Reject Rule

```bash
ansible all -m shell -a "firewall-cmd --permanent --remove-rich-rule='rule family=ipv4 source address=192.168.10.10 reject'"
```

Reload:

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

Verify:

```bash
ansible all -m shell -a "firewall-cmd --list-rich-rules"
```

---

# 12.5 Allow HTTP from a Specific IP

```bash
ansible all -m shell -a "firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.10.10 service name=http accept'"
```

This allows HTTP traffic from:

```text
192.168.10.10
```

### Explanation

```text
family=ipv4
```

IPv4 traffic.

```text
source address=192.168.10.10
```

Only traffic from this IP.

```text
service name=http
```

Applies the rule to the HTTP service.

```text
accept
```

Allows the traffic.

---

## Reload

```bash
ansible all -m shell -a "firewall-cmd --reload"
```

---

## Verify

```bash
ansible all -m shell -a "firewall-cmd --list-rich-rules"
```

---

# 13. Runtime vs Permanent

This is one of the most important concepts in firewalld.

## Runtime Rule

```bash
firewall-cmd --add-port=8080/tcp
```

The rule is applied immediately but is **not saved permanently**.

---

## Permanent Rule

```bash
firewall-cmd --add-port=8080/tcp --permanent
```

The rule is saved to the permanent configuration.

You normally need to reload firewalld:

```bash
firewall-cmd --reload
```

---

## Typical Workflow

```text
Add permanent rule
        ↓
firewall-cmd --reload
        ↓
Verify configuration
```

Example:

```bash
ansible all -b -m shell -a "firewall-cmd --permanent --add-port=8080/tcp"

ansible all -b -m shell -a "firewall-cmd --reload"

ansible all -m shell -a "firewall-cmd --list-ports"
```

---

# 14. Quick Command Reference

| Task                    | Command                                               |
| ----------------------- | ----------------------------------------------------- |
| Check installation      | `rpm -qa \| grep firewalld`                           |
| Start firewall          | `systemctl start firewalld`                           |
| Check status            | `systemctl status firewalld --no-pager`               |
| Check version           | `firewall-cmd --version`                              |
| Default zone            | `firewall-cmd --get-default-zone`                     |
| Available zones         | `firewall-cmd --get-zones`                            |
| Active zones            | `firewall-cmd --get-active-zones`                     |
| Show zone configuration | `firewall-cmd --list-all`                             |
| List services           | `firewall-cmd --list-services`                        |
| List ports              | `firewall-cmd --list-ports`                           |
| Add port                | `firewall-cmd --add-port=80/tcp --permanent`          |
| Remove port             | `firewall-cmd --remove-port=80/tcp --permanent`       |
| Reload firewall         | `firewall-cmd --reload`                               |
| Add HTTP service        | `firewall-cmd --permanent --add-service=http`         |
| Show public zone        | `firewall-cmd --zone=public --list-all`               |
| Set default zone        | `firewall-cmd --permanent --set-default-zone=private` |
| List rich rules         | `firewall-cmd --list-rich-rules`                      |

---

# 15. Common Corrections

| ❌ Incorrect               | ✅ Correct                  |
| ------------------------- | -------------------------- |
| `--virsion`               | `--version`                |
| `--get-zone`              | `--get-zones`              |
| `--get-active-zone`       | `--get-active-zones`       |
| `--zone=public--list-all` | `--zone=public --list-all` |
| `-- permanent`            | `--permanent`              |
| `privet`                  | `private`                  |
| `ipv4souce`               | `family=ipv4 source`       |
| `souce`                   | `source`                   |
| `ansible al`              | `ansible all`              |

---

# 16. Basic Practice Lab

Run these commands in order to practice:

### Step 1 — Check Firewalld

```bash
ansible all -m shell -a "rpm -qa | grep firewalld"
```

### Step 2 — Start Firewalld

```bash
ansible all -b -m shell -a "systemctl start firewalld"
```

### Step 3 — Check Status

```bash
ansible all -m shell -a "systemctl status firewalld --no-pager"
```

### Step 4 — Check Version

```bash
ansible all -m shell -a "firewall-cmd --version"
```

### Step 5 — Check Default Zone

```bash
ansible all -m shell -a "firewall-cmd --get-default-zone"
```

### Step 6 — Check Active Zones

```bash
ansible all -m shell -a "firewall-cmd --get-active-zones"
```

### Step 7 — View Configuration

```bash
ansible all -m shell -a "firewall-cmd --list-all"
```

### Step 8 — Open Port 80

```bash
ansible all -b -m shell -a "firewall-cmd --permanent --add-port=80/tcp"
```

### Step 9 — Reload Firewall

```bash
ansible all -b -m shell -a "firewall-cmd --reload"
```

### Step 10 — Verify

```bash
ansible all -m shell -a "firewall-cmd --list-ports"
```

---

# 17. Important Safety Note

⚠️ **Be careful when changing firewall rules remotely.**

A firewall configuration can block:

* SSH
* HTTP/HTTPS
* Application ports
* Network management access

Before making restrictive changes, check:

```bash
ansible all -m shell -a "firewall-cmd --get-active-zones"
```

and:

```bash
ansible all -m shell -a "firewall-cmd --list-all"
```

Make sure the SSH service and your management interface are allowed before applying firewall changes.

---

# 🚀 Useful Ansible Improvement

The examples above use the `shell` module because they demonstrate the commands directly.

For real Ansible automation, dedicated modules are generally preferable.

For example, instead of:

```bash
ansible all -m shell -a "systemctl start firewalld"
```

you can use:

```bash
ansible all -b -m service -a "name=firewalld state=started enabled=yes"
```

This is more **idempotent** and better suited for configuration management.

For larger projects, consider using Ansible's firewalld module instead of repeatedly executing `firewall-cmd` through `shell`.

---

# 📚 Summary

With Ansible and firewalld, you can remotely:

* ✅ Check whether firewalld is installed
* ✅ Start and monitor the firewall
* ✅ Check the firewalld version
* ✅ Manage firewall zones
* ✅ Open and close ports
* ✅ Open port ranges
* ✅ Allow predefined services
* ✅ Manage network interfaces
* ✅ Change the default zone
* ✅ Create rich rules
* ✅ Allow or reject traffic from specific IP addresses
* ✅ Verify firewall configuration

The basic pattern to remember is:

```text
Ansible
   ↓
Run firewall-cmd
   ↓
Modify firewall configuration
   ↓
Reload firewalld
   ↓
Verify the configuration
```

🔥 **Happy Learning & Secure Your Linux Servers!**
