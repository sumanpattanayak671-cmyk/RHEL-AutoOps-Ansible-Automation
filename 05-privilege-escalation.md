# Privilege Escalation

Use `become` when a task needs root permissions, for example when managing packages, services, SELinux policy, firewall rules, or mounts.

| Option | Purpose |
| --- | --- |
| `-b` / `--become` | Enable privilege escalation |
| `-K` / `--ask-become-pass` | Prompt for the sudo password |
| `--become-user root` | Select the target user |
| `--become-method sudo` | Use sudo for escalation |

Example:

```bash
ansible all -b -m ansible.builtin.command -a 'id'
```

Enable escalation by default in `ansible.cfg`:

```ini
[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

Disable it for a particular play or task:

```yaml
become: false
```
