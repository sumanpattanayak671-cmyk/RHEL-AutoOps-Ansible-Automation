# Firewall and Scheduling

## Firewalld

Install and start the firewall service:

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

Check the firewall:

```bash
ansible all -b -m ansible.builtin.command -a 'firewall-cmd --get-active-zones'
ansible all -b -m ansible.builtin.command -a 'firewall-cmd --list-services'
ansible all -b -m ansible.builtin.command -a 'firewall-cmd --list-ports'
```

## Cron jobs

Use `ansible.builtin.cron` for recurring tasks:

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

Use `at` for one-time jobs, and cron or systemd timers for recurring jobs.
