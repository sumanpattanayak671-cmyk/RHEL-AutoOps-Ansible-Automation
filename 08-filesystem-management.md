# Filesystem Management

Use Ansible modules instead of editing `/etc/fstab` manually.

## Persistent mount playbook

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

Verify the mount:

```bash
ansible all -m ansible.builtin.command -a 'findmnt /data'
ansible all -m ansible.builtin.command -a 'df -Th /data'
```

> Always confirm the target device. Formatting or changing partitions can destroy data.
