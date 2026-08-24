# Web, Mail, and Update Playbook

This playbook installs Apache on web servers, installs Postfix on application servers, and applies available updates to all servers.

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

Run a dry run before applying changes:

```bash
ansible-playbook intranet.yml --check --diff
ansible-playbook intranet.yml
```

> Test updates first because they can change running services and may require a reboot.
