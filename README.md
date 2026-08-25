# Ansible Administration Lab Guide

An organized RHEL 9 / Ansible learning repository.

## Topics

1. [Lab Topology and Control Node Setup](01-topology-and-control-node.md)
2. [Inventory and Configuration](02-inventory-and-configuration.md)
3. [SSH Authentication](03-ssh-authentication.md)
4. [Web, Mail, and Update Playbook](04-intranet-playbook.md)
5. [Privilege Escalation](05-privilege-escalation.md)
6. [Common Ansible Modules](06-common-modules.md)
7. [Package and Service Management](07-packages-and-services.md)
8. [Filesystem Management](08-filesystem-management.md)
9. [SELinux](09-selinux.md)
10. [Firewall and Scheduling](10-firewall-and-scheduling.md)
11. [Troubleshooting](11-troubleshooting.md)

> Use SSH keys and Ansible Vault for real environments. Do not commit credentials, private keys, or vault passwords to GitHub.

---------------------------------------------------------
# Ansible Administration Lab Guide

This repository documents a practical Ansible lab on RHEL-compatible systems. It covers inventory design, control-node setup, SSH authentication, privilege escalation, ad-hoc modules, package and service management, SELinux, firewalld, and scheduling.

> **Lab assumptions:** Commands target RHEL 9 or a compatible distribution. Run destructive storage commands only on disposable lab disks.

## Contents

- [Lab topology](#lab-topology)
- [Install and configure the control node](#install-and-configure-the-control-node)
- [Inventory](#inventory)
- [SSH authentication](#ssh-authentication)
- [Playbook: web, mail, and updates](#playbook-web-mail-and-updates)
- [Privilege escalation](#privilege-escalation)
- [Useful modules](#useful-modules)
- [Package and service management](#package-and-service-management)
- [SELinux](#selinux)
- [Firewall management](#firewall-management)
- [Scheduling](#scheduling)
- [Troubleshooting](#troubleshooting)

## Lab topology

| Role | Hostname | Address | Gateway |
| --- | --- | --- | --- |
| Control node | `ansible-server.india.com` | `192.168.10.1/24` | `192.168.10.1` |
| Managed node 1 | `node1.india.com` | `192.168.10.2/24` | `192.168.10.1` |
| Managed node 2 | `node2.india.com` | `192.168.10.3/24` | `192.168.10.1` |

Configure hostnames:

```bash
sudo hostnamectl set-hostname ansible-server.india.com
sudo hostnamectl set-hostname node1.india.com
sudo hostnamectl set-hostname node2.india.com
```

Add the following entries to `/etc/hosts` on all three machines:

```text
192.168.10.1 ansible-server.india.com ansible-server
192.168.10.2 node1.india.com node1
192.168.10.3 node2.india.com node2
```

Verify hostname resolution:

```bash
getent hosts ansible-server node1 node2
```

## Install and configure the control node

Install Ansible Core on the control node:

```bash
sudo dnf install -y ansible-core
ansible --version
```

Create an Ansible user on the control node and managed nodes:

```bash
sudo useradd -m ansible
sudo passwd ansible
```

Give the `ansible` user sudo permissions on the managed nodes:

```bash
echo 'ansible ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
sudo visudo -cf /etc/sudoers.d/ansible
```

> For production, grant only the minimum required sudo permissions.

## Project structure

Create a project directory:

```text
automation/
├── ansible.cfg
├── inventory.ini
└── intranet.yml
```

Create `ansible.cfg`:

```ini
[defaults]
inventory = ./inventory.ini
remote_user = ansible
host_key_checking = True
interpreter_python = auto_silent

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

For a temporary lab only, you can use:

```ini
host_key_checking = False
```

## Inventory

Create `inventory.ini`:

```ini
[webservers]
web1.example.com
web2.example.com

[appservers]
host1.example.com
host2.example.com

[servers:children]
webservers
appservers
```

For the two-node lab environment, use:

```ini
[webservers]
node1.india.com

[appservers]
node2.india.com

[servers:children]
webservers
appservers
```

Verify inventory configuration:

```bash
ansible-inventory --graph
ansible all --list-hosts
ansible all -m ansible.builtin.ping
```

## System-wide inventory

The default Ansible inventory file is:

```text
/etc/ansible/hosts
```

Example:

```ini
[web]
node1
node2
```

To configure a custom default inventory, edit `/etc/ansible/ansible.cfg`:

```ini
[defaults]
inventory = /home/ansible/automation/inventory.ini
```

## SSH authentication

### SSH key-based authentication

SSH keys are the recommended authentication method.

On the control node:

```bash
su - ansible
ssh-keygen -t ed25519 -C 'ansible-control-node'
ssh-copy-id ansible@node1
ssh-copy-id ansible@node2
```

Test SSH access:

```bash
ssh ansible@node1 'id'
ssh ansible@node2 'id'
```

Test Ansible connectivity:

```bash
ansible all -m ansible.builtin.ping
```

### Password-based authentication

Install `sshpass` only if password authentication is required for a lab:

```bash
sudo dnf install -y sshpass
ansible all -m ansible.builtin.ping --ask-pass
```

On managed nodes, ensure this setting is enabled in `/etc/ssh/sshd_config`:

```text
PasswordAuthentication yes
```

Restart SSH after making changes:

```bash
sudo systemctl restart sshd
```

> Never commit SSH passwords to Git repositories. Use SSH keys or Ansible Vault.

## Playbook: web, mail, and updates

Create `intranet.yml`:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache HTTP Server
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Enable and start Apache
      ansible.builtin.systemd:
        name: httpd
        state: started
        enabled: true

- name: Configure application servers
  hosts: appservers
  become: true

  tasks:
    - name: Install Postfix
      ansible.builtin.dnf:
        name: postfix
        state: present

    - name: Enable and start Postfix
      ansible.builtin.systemd:
        name: postfix
        state: started
        enabled: true

- name: Apply available package updates
  hosts: servers
  become: true

  tasks:
    - name: Update installed packages
      ansible.builtin.dnf:
        name: '*'
        state: latest
        update_only: true
```

Run the playbook:

```bash
ansible-playbook intranet.yml --check --diff
ansible-playbook intranet.yml
```

## Privilege escalation

Use privilege escalation when tasks require root permissions.

| Option | Purpose |
| --- | --- |
| `-b` or `--become` | Enable privilege escalation |
| `-K` or `--ask-become-pass` | Ask for the sudo password |
| `--become-user root` | Become the root user |
| `--become-method sudo` | Use sudo for escalation |

Example:

```bash
ansible all -b -m ansible.builtin.command -a 'id'
```

## Useful modules

| Module | Purpose |
| --- | --- |
| `file` | Create files, directories, links, and manage permissions |
| `copy` | Copy files or content to managed nodes |
| `fetch` | Copy files from managed nodes to the control node |
| `command` | Run commands without shell features |
| `shell` | Run commands with pipes, redirects, and variables |
| `stat` | Check file metadata |
| `lineinfile` | Add, remove, or modify a single line |
| `replace` | Replace matching text |
| `user` | Manage users |
| `group` | Manage groups |
| `systemd` | Manage services |
| `mount` | Manage mounted filesystems |

### File module

```bash
ansible all -b -m ansible.builtin.file -a 'path=/tmp/redhat state=directory mode=0755'

ansible all -m ansible.builtin.file -a 'path=/tmp/file20.txt state=touch'

ansible all -m ansible.builtin.file -a 'path=/tmp/redhat.txt state=absent'

ansible all -m ansible.builtin.file -a 'src=/tmp/file100.txt dest=/tmp/link1 state=link'
```

### Command module

```bash
ansible all -m ansible.builtin.command -a 'uptime'

ansible all -m ansible.builtin.command -a 'id ansible'

ansible all -m ansible.builtin.command -a 'ls -l /tmp'
```

### Shell module

Use the shell module only when shell features are required.

```bash
ansible all -m ansible.builtin.shell -a 'cut -d: -f1 /etc/passwd'

ansible all -m ansible.builtin.shell -a 'df -Th'

ansible all -m ansible.builtin.shell -a 'lsblk -f'
```

### Copy module

```bash
ansible all -m ansible.builtin.copy -a 'src=file1.txt dest=/tmp/file1.txt'

ansible all -m ansible.builtin.copy -a 'content=GOOD dest=/tmp/file100.txt mode=0644'

ansible all -b -m ansible.builtin.copy -a 'src=index.html dest=/var/www/html/index.html mode=0644'
```

### Fetch module

```bash
ansible node1 -m ansible.builtin.fetch -a 'src=/tmp/file100.txt dest=backup/ flat=true'
```

### Stat module

```bash
ansible all -m ansible.builtin.stat -a 'path=/tmp/file100.txt'
```

### Lineinfile module

```bash
ansible all -m ansible.builtin.lineinfile -a 'path=/tmp/file100.txt line=Content create=true'

ansible all -m ansible.builtin.lineinfile -a 'path=/tmp/file100.txt line=Content state=absent'
```

### Replace module

```bash
ansible all -m ansible.builtin.replace -a 'path=/tmp/file100.txt regexp=AFTERNOON replace=MORNING'
```

### User and group modules

```bash
ansible all -b -m ansible.builtin.group -a 'name=HR state=present'

ansible all -b -m ansible.builtin.user -a 'name=developer state=present'

ansible all -b -m ansible.builtin.user -a 'name=developer groups=HR append=true'

ansible all -b -m ansible.builtin.user -a 'name=developer state=absent remove=true'
```

## Package and service management

Install packages:

```bash
ansible all -b -m ansible.builtin.dnf -a 'name=httpd state=present'

ansible all -b -m ansible.builtin.package -a 'name=vsftpd state=present'
```

Manage Apache:

```bash
ansible all -b -m ansible.builtin.systemd -a 'name=httpd state=started enabled=true'

ansible all -m ansible.builtin.command -a 'systemctl is-active httpd'

ansible all -m ansible.builtin.uri -a 'url=http://localhost status_code=200'
```

Stop and disable a service:

```bash
ansible all -b -m ansible.builtin.systemd -a 'name=httpd state=stopped enabled=false'
```

## Persistent filesystem mount

Use the mount module instead of manually editing `/etc/fstab`.

```yaml
---
- name: Mount application data
  hosts: servers
  become: true

  tasks:
    - name: Create mount point
      ansible.builtin.file:
        path: /data
        state: directory
        mode: '0755'

    - name: Mount and persist the filesystem
      ansible.posix.mount:
        path: /data
        src: /dev/nvme0n2p1
        fstype: ext4
        opts: defaults
        state: mounted
```

> Confirm the correct device before formatting or mounting a disk.

## SELinux

SELinux is a mandatory access-control security system.

| Mode | Description |
| --- | --- |
| Enforcing | Enforces SELinux policy and blocks unauthorized access |
| Permissive | Logs policy violations but does not block access |
| Disabled | Disables SELinux after reboot; avoid in production |

Check SELinux status:

```bash
sestatus
getenforce
ansible all -m ansible.builtin.command -a 'sestatus'
```

Configure an Apache web directory outside `/var/www/html`:

```bash
sudo mkdir -p /web
sudo dnf install -y httpd policycoreutils-python-utils

sudo semanage fcontext -a -t httpd_sys_content_t '/web(/.*)?'
sudo restorecon -Rv /web

sudo systemctl enable --now httpd
```

Verify SELinux labels:

```bash
ls -lZ /web
ls -lZ /var/www/html
```

List SELinux booleans:

```bash
getsebool -a
```

Enable an SELinux boolean permanently:

```bash
sudo setsebool -P httpd_can_network_connect on
```

## Firewall management

Ensure firewalld is installed and running:

```bash
sudo dnf install -y firewalld
sudo systemctl enable --now firewalld
```

Allow HTTP using Ansible:

```yaml
---
- name: Allow web traffic
  hosts: webservers
  become: true

  tasks:
    - name: Enable HTTP service in firewalld
      ansible.posix.firewalld:
        service: http
        permanent: true
        immediate: true
        state: enabled
```

Check firewall settings:

```bash
ansible all -b -m ansible.builtin.command -a 'firewall-cmd --get-active-zones'

ansible all -b -m ansible.builtin.command -a 'firewall-cmd --list-services'

ansible all -b -m ansible.builtin.command -a 'firewall-cmd --list-ports'
```

## Scheduling

Use Ansible’s cron module for recurring tasks.

```yaml
---
- name: Add a disk usage report job
  hosts: servers

  tasks:
    - name: Write disk usage report every day at 17:45
      ansible.builtin.cron:
        name: disk-usage-report
        minute: '45'
        hour: '17'
        job: 'df -h >> /home/ansible/cron-disk-monitor.log 2>&1'
```

For one-time scheduled jobs, use the `at` command. For recurring jobs, use cron or systemd timers.

## Troubleshooting

```bash
# View inventory structure
ansible-inventory --graph

# List target hosts
ansible all --list-hosts

# Test connectivity with detailed output
ansible all -m ansible.builtin.ping -vvv

# Check remote identity
ansible all -m ansible.builtin.command -a 'id'

# Check sudo access
ansible all -b -m ansible.builtin.command -a 'id'

# Gather basic facts
ansible all -m ansible.builtin.setup -a 'filter=ansible_distribution*'

# Check Apache service status
ansible webservers -b -m ansible.builtin.systemd -a 'name=httpd'
```

Common causes of Ansible failures include:

- Incorrect hostname resolution
- Wrong SSH username, key, or password
- Missing sudo permissions
- Host-key verification problems
- Incorrect inventory file path
- SELinux blocking access
- Firewalld blocking network traffic
