# Lab Topology and Control Node Setup

## Topology

| Role | Hostname | Address | Gateway |
| --- | --- | --- | --- |
| Control node | `ansible-server.india.com` | `192.168.10.1/24` | `192.168.10.1` |
| Managed node 1 | `node1.india.com` | `192.168.10.2/24` | `192.168.10.1` |
| Managed node 2 | `node2.india.com` | `192.168.10.3/24` | `192.168.10.1` |

Set the appropriate hostname on each server:

```bash
sudo hostnamectl set-hostname <hostname>
```

Add these records to `/etc/hosts` on all machines:

```text
192.168.10.1 ansible-server.india.com ansible-server
192.168.10.2 node1.india.com node1
192.168.10.3 node2.india.com node2
```

Verify resolution:

```bash
getent hosts ansible-server node1 node2
```

## Install Ansible

On the control node:

```bash
sudo dnf install -y ansible-core
ansible --version
```

Create an automation account on the control node and both managed nodes:

```bash
sudo useradd -m ansible
sudo passwd ansible
```

For the lab, configure passwordless sudo on managed nodes:

```bash
echo 'ansible ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
sudo visudo -cf /etc/sudoers.d/ansible
```

> Scope sudo access narrowly in production.
