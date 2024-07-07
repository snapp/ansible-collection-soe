# Ⓐ Ansible Role: local_users

Manage local user accounts.

## Using this role in an Ansible playbook

The following example shows how this role can be defined in a playbook with parameters passed to override default variables.

> [!INFORMATION]
> It is recommended that you use a Fully Qualified Collection Name (FQCN) when referencing your roles.

```yaml
---
- hosts: 'all'
  roles:
    - role: infra.soe.local_users
      local_users:
        - name: example
          uid: 1234
          group: example
          gid: 1234
          groups: ''
          comment: 'Example User'
          home: '/home'
          sudo: 'ALL=(ALL)  ALL'
          ssh_public_key: 'ssh-rsa AAAAB3NzaC1yc2EAmEQ== Example User'
      tags: infra.soe.local_users
```

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
