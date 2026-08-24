# Common Ansible Modules

Prefer Ansible modules over raw shell commands because modules are safer and idempotent.

| Module | Purpose |
| --- | --- |
| `file` | Files, directories, permissions, and links |
| `copy` | Copy local files or create remote content |
| `fetch` | Retrieve files from managed nodes |
| `command` | Run commands without shell parsing |
| `shell` | Run commands requiring pipes or redirection |
| `stat` | Inspect file metadata |
| `lineinfile` | Manage one line in a file |
| `replace` | Replace matching content |
| `user` / `group` | Manage accounts and groups |

## File module

```bash
ansible all -b -m ansible.builtin.file -a 'path=/tmp/redhat state=directory mode=0755'
ansible all -m ansible.builtin.file -a 'path=/tmp/file20.txt state=touch'
ansible all -m ansible.builtin.file -a 'path=/tmp/redhat.txt state=absent'
ansible all -m ansible.builtin.file -a 'src=/tmp/file100.txt dest=/tmp/link1 state=link'
```

## Command and shell modules

```bash
ansible all -m ansible.builtin.command -a 'uptime'
ansible all -m ansible.builtin.command -a 'id ansible'
ansible all -m ansible.builtin.shell -a 'cut -d: -f1 /etc/passwd'
ansible all -m ansible.builtin.shell -a 'df -Th'
```

Use `shell` only when shell syntax is necessary.

## Copy, fetch, and stat

```bash
ansible all -m ansible.builtin.copy -a 'src=file1.txt dest=/tmp/file1.txt'
ansible all -m ansible.builtin.copy -a 'content=GOOD dest=/tmp/file100.txt mode=0644'
ansible node1 -m ansible.builtin.fetch -a 'src=/tmp/file100.txt dest=backup/ flat=true'
ansible all -m ansible.builtin.stat -a 'path=/tmp/file100.txt'
```

## Text management

```bash
ansible all -m ansible.builtin.lineinfile -a 'path=/tmp/file100.txt line=Content create=true'
ansible all -m ansible.builtin.replace -a 'path=/tmp/file100.txt regexp=AFTERNOON replace=MORNING'
```

## Users and groups

```bash
ansible all -b -m ansible.builtin.group -a 'name=HR state=present'
ansible all -b -m ansible.builtin.user -a 'name=developer state=present'
ansible all -b -m ansible.builtin.user -a 'name=developer groups=HR append=true'
ansible all -b -m ansible.builtin.user -a 'name=developer state=absent remove=true'
```
