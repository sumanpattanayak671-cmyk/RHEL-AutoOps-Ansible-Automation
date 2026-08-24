# Package and Service Management

On RHEL 9, use `ansible.builtin.dnf` for package management.

## Package management

```bash
ansible all -b -m ansible.builtin.dnf -a 'name=httpd state=present'
ansible all -b -m ansible.builtin.package -a 'name=vsftpd state=present'
ansible all -b -m ansible.builtin.dnf -a 'name=httpd state=absent'
```

## Service management

Start and enable Apache:

```bash
ansible all -b -m ansible.builtin.systemd -a 'name=httpd state=started enabled=true'
```

Stop and disable Apache:

```bash
ansible all -b -m ansible.builtin.systemd -a 'name=httpd state=stopped enabled=false'
```

Check the service:

```bash
ansible all -m ansible.builtin.command -a 'systemctl is-active httpd'
```

## Deploy a test page

```bash
ansible webservers -b -m ansible.builtin.copy -a 'content=Hello from Ansible dest=/var/www/html/index.html mode=0644'
ansible webservers -m ansible.builtin.uri -a 'url=http://localhost status_code=200'
```
