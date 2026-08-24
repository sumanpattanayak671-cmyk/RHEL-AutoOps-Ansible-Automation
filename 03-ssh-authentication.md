# SSH Authentication

## SSH key-based authentication

SSH keys are the recommended authentication method.

On the control node, as the `ansible` user:

```bash
ssh-keygen -t ed25519 -C 'ansible-control-node'
ssh-copy-id ansible@node1
ssh-copy-id ansible@node2
```

Test connectivity:

```bash
ssh ansible@node1 'id'
ssh ansible@node2 'id'
ansible all -m ansible.builtin.ping
```

## Password-based authentication

Install `sshpass` only where password authentication is required for a lab:

```bash
sudo dnf install -y sshpass
ansible all -m ansible.builtin.ping --ask-pass
```

On a managed node, confirm this setting in `/etc/ssh/sshd_config`:

```text
PasswordAuthentication yes
```

Restart SSH after making a change:

```bash
sudo systemctl restart sshd
```

## Secrets

Never place passwords in a Git repository. Store secrets with Ansible Vault:

```bash
ansible-vault create group_vars/all/vault.yml
ansible-vault edit group_vars/all/vault.yml
ansible-playbook intranet.yml --ask-vault-pass
```
