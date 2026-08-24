# Inventory and Configuration

## Project layout

```text
automation/
├── ansible.cfg
├── inventory.ini
└── intranet.yml
```

## Project configuration

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

For a temporary lab, you may set `host_key_checking = False`. Keep host key verification enabled in real environments.

## Inventory groups

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

For the two-node lab:

```ini
[webservers]
node1.india.com

[appservers]
node2.india.com

[servers:children]
webservers
appservers
```

Verify the inventory:

```bash
ansible-inventory --graph
ansible all --list-hosts
ansible all -m ansible.builtin.ping
```

## System-wide inventory

The default inventory is `/etc/ansible/hosts`. To use a custom default inventory, add this to `/etc/ansible/ansible.cfg`:

```ini
[defaults]
inventory = /home/ansible/automation/inventory.ini
```
