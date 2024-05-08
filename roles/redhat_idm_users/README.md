# Ⓐ Ansible Role: redhat_idm_users

Manage IdM user accounts.

## Using this role in an Ansible playbook

The following example shows how this role can be defined in a playbook with parameters passed to override default variables.

> [!INFORMATION]
> It is recommended that you use a Fully Qualified Collection Name (FQCN) when referencing your roles.

```yaml
---
- hosts: "{{ groups['idm_server'][0] }}"
  roles:
    - role: infra.soe.redhat_idm_users
      redhat_idm_users:
        - description: Satellite Automation Account
          name:
            username: satellite
            first: Satellite
            last: Automation
            full: Satellite Automation Account
          initials: SAA
          uid: 800
          groups: ''
          home: /var/lib
          password: |
            $6$rounds=100000$redacted
          ssh_public_key: |
            ssh-rsa AAAAredacted
        - name:
            username: example
            first: Example
            last: User
            full: Example User
          initials: EU
          email: user@example.com
      tags: infra.soe.redhat_idm_users
```

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
