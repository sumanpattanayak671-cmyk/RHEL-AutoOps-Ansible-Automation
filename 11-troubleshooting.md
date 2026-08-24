# Troubleshooting

## Inventory and connectivity

```bash
ansible-inventory --graph
ansible all --list-hosts
ansible all -m ansible.builtin.ping -vvv
```

## Identity and sudo

```bash
ansible all -m ansible.builtin.command -a 'id'
ansible all -b -m ansible.builtin.command -a 'id'
```

## Facts and services

```bash
ansible all -m ansible.builtin.setup -a 'filter=ansible_distribution*'
ansible webservers -b -m ansible.builtin.systemd -a 'name=httpd'
```

## Common causes of failures

- Incorrect hostname resolution or `/etc/hosts` entries
- Wrong SSH user, key, password, or permissions
- Missing sudo permission for the automation user
- Host-key verification mismatch
- Incorrect inventory file path
- SELinux labels or booleans blocking a service
- Firewalld blocking required traffic
