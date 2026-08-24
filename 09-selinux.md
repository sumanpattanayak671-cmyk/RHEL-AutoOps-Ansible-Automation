# SELinux

SELinux is a mandatory access-control system in RHEL.

| Mode | Description |
| --- | --- |
| Enforcing | Enforces policy and blocks unauthorized access |
| Permissive | Logs policy violations without blocking access |
| Disabled | Turns SELinux off after a reboot; avoid in production |

## Check SELinux

```bash
sestatus
getenforce
ansible all -m ansible.builtin.command -a 'sestatus'
```

## Label a custom Apache web directory

```bash
sudo mkdir -p /web
sudo dnf install -y httpd policycoreutils-python-utils
sudo semanage fcontext -a -t httpd_sys_content_t '/web(/.*)?'
sudo restorecon -Rv /web
sudo systemctl enable --now httpd
```

Verify labels:

```bash
ls -lZ /web
ls -lZ /var/www/html
```

## SELinux booleans

```bash
getsebool -a
sudo setsebool -P httpd_can_network_connect on
```

Use `chcon` only for temporary testing because `restorecon` removes labels applied with `chcon`.
