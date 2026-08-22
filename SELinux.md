# SELinux Configuration and Ansible Automation Guide

This guide demonstrates how to configure **SELinux manually** and automate common SELinux tasks using **Ansible** on RHEL-based systems.

---

## Table of Contents

* [1. SELinux Overview](#1-selinux-overview)
* [2. Manual SELinux Configuration](#2-manual-selinux-configuration)

  * [2.1 Check SELinux Status](#21-check-selinux-status)
  * [2.2 Create a Web Directory](#22-create-a-web-directory)
  * [2.3 Install and Configure Apache](#23-install-and-configure-apache)
  * [2.4 Configure SELinux Context](#24-configure-selinux-context)
  * [2.5 Make SELinux Context Permanent](#25-make-selinux-context-permanent)
  * [2.6 Configure SELinux Booleans](#26-configure-selinux-booleans)
  * [2.7 Verify the Web Server](#27-verify-the-web-server)
* [3. SELinux Automation with Ansible](#3-selinux-automation-with-ansible)

  * [3.1 Check SELinux Status](#31-check-selinux-status)
  * [3.2 Change SELinux Mode](#32-change-selinux-mode)
  * [3.3 Create and Check SELinux File Context](#33-create-and-check-selinux-file-context)
  * [3.4 Install Apache](#34-install-apache)
  * [3.5 Create Web Content](#35-create-web-content)
  * [3.6 Configure SELinux Context](#36-configure-selinux-context)
  * [3.7 Configure Permanent File Context](#37-configure-permanent-file-context)
  * [3.8 Check Apache SELinux Process Context](#38-check-apache-selinux-process-context)
  * [3.9 Configure SELinux Port Context](#39-configure-selinux-port-context)
* [4. Useful SELinux Commands](#4-useful-selinux-commands)

---

# 1. SELinux Overview

**SELinux (Security-Enhanced Linux)** provides an additional security layer on Linux systems.

SELinux uses **security contexts, policies, types, and booleans** to control what processes are allowed to access.

### SELinux Modes

| Mode         | Description                                  |
| ------------ | -------------------------------------------- |
| `Enforcing`  | SELinux policy is actively enforced          |
| `Permissive` | Policy violations are logged but not blocked |
| `Disabled`   | SELinux is completely disabled               |

### Check Current SELinux Mode

```bash
getenforce
```

Example:

```text
Enforcing
```

---

# 2. Manual SELinux Configuration

## 2.1 Check SELinux Status

Check the complete SELinux status:

```bash
sestatus
```

Check the current enforcement mode:

```bash
getenforce
```

Check the SELinux configuration:

```bash
cat /etc/selinux/config
```

Check the loaded SELinux policy:

```bash
sestatus | grep "Loaded policy name"
```

Check the SELinux context of a file:

```bash
ls -lZ test.sh
```

---

## 2.2 Create a Web Directory

Switch to the root user:

```bash
su - root
```

Create the web directory:

```bash
mkdir -p /web
```

Move into the directory:

```bash
cd /web
```

Create the web page:

```bash
vim index.html
```

Example content:

```html
<h1>SELinux Apache Test Page</h1>
<p>This page is hosted from /web.</p>
```

Check the SELinux context:

```bash
ls -Zl /web/index.html
```

Check the directory context:

```bash
ls -Zld /web
```

---

# 2.3 Install and Configure Apache

Install Apache:

```bash
dnf install httpd -y
```

Check the Apache configuration:

```bash
httpd -t
```

Open the Apache configuration:

```bash
vim /etc/httpd/conf/httpd.conf
```

Configure the document root:

```apache
DocumentRoot "/web"
```

> **Important:** `DocumentRoot` should point to the directory containing the web files, not directly to `index.html`.

Restart Apache:

```bash
systemctl restart httpd
```

Enable Apache at boot:

```bash
systemctl enable httpd
```

Or enable and start it immediately:

```bash
systemctl enable --now httpd
```

Check Apache status:

```bash
systemctl status httpd --no-pager
```

Verify the configuration:

```bash
apachectl configtest
```

---

# 2.4 Configure SELinux Context

Apache needs the correct SELinux type to access files used as web content.

Check the existing context:

```bash
ls -Zl /web/index.html
```

Set the web content type temporarily:

```bash
chcon -t httpd_sys_content_t /web/index.html
```

Set the directory context:

```bash
chcon -R -t httpd_sys_content_t /web
```

Verify:

```bash
ls -Zl /web
```

```bash
ls -Zld /web
```

Restart Apache:

```bash
systemctl restart httpd
```

---

# 2.5 Make SELinux Context Permanent

`chcon` changes the current SELinux context, but the change may be lost after a filesystem relabel or `restorecon`.

For a permanent SELinux file-context rule, use `semanage`.

Install the required SELinux management tools:

```bash
dnf install policycoreutils-python-utils -y
```

Check whether `semanage` is available:

```bash
command -v semanage
```

Add a permanent SELinux file-context rule:

```bash
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
```

Apply the rule:

```bash
restorecon -Rv /web
```

Verify:

```bash
ls -Zl /web
```

Check the configured rule:

```bash
semanage fcontext -l | grep "/web"
```

### Remove the Permanent Rule

```bash
semanage fcontext -d -t httpd_sys_content_t "/web(/.*)?"
```

Then restore the default context:

```bash
restorecon -Rv /web
```

---

# 2.6 Configure SELinux Booleans

SELinux booleans allow certain policy features to be enabled or disabled.

### List All SELinux Booleans

```bash
getsebool -a
```

### Enable a Boolean Temporarily

```bash
setsebool <boolean_name> on
```

Example:

```bash
setsebool httpd_can_network_connect on
```

### Enable a Boolean Permanently

```bash
setsebool -P <boolean_name> on
```

Example:

```bash
setsebool -P httpd_can_network_connect on
```

### Check a Specific Boolean

```bash
getsebool httpd_can_network_connect
```

---

# 2.7 Verify the Web Server

Check Apache status:

```bash
systemctl status httpd --no-pager
```

Test the Apache configuration:

```bash
apachectl configtest
```

Test the web server locally:

```bash
curl localhost
```

Expected output should contain the content of:

```text
/web/index.html
```

---

# 3. SELinux Automation with Ansible

The following commands can be executed from the **Ansible control node** to manage multiple RHEL-based servers.

---

## 3.1 Check SELinux Status

Check SELinux status on all managed hosts:

```bash
ansible all -m command -a 'sestatus'
```

Check the current SELinux mode:

```bash
ansible all -m command -a 'getenforce'
```

Check the SELinux configuration:

```bash
ansible all -m command -a 'cat /etc/selinux/config'
```

Check the loaded SELinux policy:

```bash
ansible all -m shell -a 'sestatus | grep "Loaded policy name"'
```

---

## 3.2 Change SELinux Mode

Change SELinux to **Permissive mode temporarily**:

```bash
ansible all -m ansible.posix.selinux -a 'state=permissive'
```

> This changes the runtime SELinux mode. It does not permanently disable SELinux.

Check the result:

```bash
ansible all -m command -a 'getenforce'
```

Expected:

```text
Permissive
```

Return SELinux to Enforcing mode:

```bash
ansible all -m ansible.posix.selinux -a 'state=enforcing'
```

---

## 3.3 Create and Check SELinux File Context

Create a test file:

```bash
ansible all -m shell -a 'touch /tmp/test_selinux.txt'
```

Check its SELinux context:

```bash
ansible all -m command -a 'ls -lZ /tmp/test_selinux.txt'
```

---

## 3.4 Install Apache

Check the existing Apache document directory:

```bash
ansible all -m command -a 'ls -lZ /var/www/html'
```

Install Apache:

```bash
ansible all -m dnf -a 'name=httpd state=present'
```

---

## 3.5 Create Web Content

Create the web page:

```bash
ansible all -m shell -a 'echo "THIS IS THE ANSIBLE TEST PAGE" > /var/www/html/index.html'
```

Check the SELinux context:

```bash
ansible all -m shell -a 'ls -Zl /var/www/html/index.html'
```

---

## 3.6 Configure SELinux Context

Set the SELinux web-content type temporarily:

```bash
ansible all -m shell -a 'chcon -t httpd_sys_content_t /var/www/html/index.html'
```

Verify:

```bash
ansible all -m shell -a 'ls -Zl /var/www/html/index.html'
```

Restart and enable Apache:

```bash
ansible all -m shell -a 'systemctl enable --now httpd'
```

Test the web server:

```bash
ansible all -m shell -a 'curl localhost'
```

---

## 3.7 Configure Permanent File Context

Check whether `semanage` is installed:

```bash
ansible all -m shell -a 'command -v semanage'
```

Install the required package if necessary:

```bash
ansible all -m dnf -a 'name=policycoreutils-python-utils state=present'
```

Create a permanent SELinux file-context rule for Apache's default document root:

```bash
ansible all -m shell -a 'semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"'
```

Apply the context:

```bash
ansible all -m shell -a 'restorecon -Rv /var/www/html'
```

Verify:

```bash
ansible all -m shell -a 'ls -Zl /var/www/html/index.html'
```

### Remove the Permanent Rule

```bash
ansible all -m shell -a 'semanage fcontext -d -t httpd_sys_content_t "/var/www/html(/.*)?"'
```

Then restore the default context:

```bash
ansible all -m shell -a 'restorecon -Rv /var/www/html'
```

---

## 3.8 Check Apache SELinux Process Context

To view the SELinux context of the Apache processes:

```bash
ansible all -m shell -a 'ps -eZ | grep httpd'
```

Apache processes should normally run under an appropriate SELinux domain such as:

```text
httpd_t
```

---

# 3.9 Configure SELinux Port Context

List SELinux port contexts:

```bash
ansible all -m shell -a 'semanage port -l | grep http_port_t'
```

Add TCP port `8085` to the HTTP SELinux port type:

```bash
ansible all -m shell -a 'semanage port -a -t http_port_t -p tcp 8085'
```

Verify:

```bash
ansible all -m shell -a 'semanage port -l | grep http_port_t'
```

### Remove Port Context

```bash
ansible all -m shell -a 'semanage port -d -t http_port_t -p tcp 8085'
```

> If the port already exists in SELinux policy, use `-m` instead of `-a` to modify its type.

---

# 4. Useful SELinux Commands

## Check SELinux Status

```bash
sestatus
```

## Check Current Mode

```bash
getenforce
```

## View SELinux Configuration

```bash
cat /etc/selinux/config
```

## View File SELinux Context

```bash
ls -Z
```

## View Directory SELinux Context

```bash
ls -Zd /web
```

## View SELinux Booleans

```bash
getsebool -a
```

## Enable Boolean Temporarily

```bash
setsebool <boolean_name> on
```

## Enable Boolean Permanently

```bash
setsebool -P <boolean_name> on
```

## Change File Context Temporarily

```bash
chcon -t httpd_sys_content_t <file>
```

## Restore SELinux Context

```bash
restorecon -Rv <directory>
```

## Add Permanent File Context

```bash
semanage fcontext -a -t <SELinux_type> "<path>(/.*)?"
```

## List File Context Rules

```bash
semanage fcontext -l
```

## List SELinux Port Contexts

```bash
semanage port -l
```

## Add a Port

```bash
semanage port -a -t http_port_t -p tcp <port>
```

## Remove a Port

```bash
semanage port -d -t http_port_t -p tcp <port>
```

## Check Process Context

```bash
ps -eZ
```

## Check Apache Process Context

```bash
ps -eZ | grep httpd
```

---

# 5. Important Notes

### `chcon` vs `semanage fcontext`

`chcon` changes the SELinux context directly:

```bash
chcon -t httpd_sys_content_t /web/index.html
```

However, it is **not the recommended permanent solution**.

For permanent configuration, use:

```bash
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
```

### Why `restorecon` is Important

`restorecon` applies the SELinux context defined by the SELinux file-context database.

```bash
restorecon -Rv /web
```

### Apache DocumentRoot

The Apache `DocumentRoot` should point to the **directory** containing the website:

```apache
DocumentRoot "/web"
```

Not:

```apache
DocumentRoot "/web/index.html"
```

---

# 6. Quick Verification

After completing the configuration, run:

```bash
getenforce
```

```bash
ls -Zl /web
```

```bash
semanage fcontext -l | grep "/web"
```

```bash
systemctl status httpd --no-pager
```

```bash
apachectl configtest
```

```bash
curl localhost
```

If everything is configured correctly, SELinux should remain **Enforcing**, Apache should be running, and the web page should be accessible.

---

## Author

**Linux / Ansible / SELinux Practical Notes**

Topics covered:

* SELinux modes
* SELinux security contexts
* SELinux booleans
* `chcon`
* `restorecon`
* `semanage fcontext`
* SELinux port contexts
* Apache SELinux configuration
* Ansible SELinux automation
* RHEL Apache automation
