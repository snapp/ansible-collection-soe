# Ⓐ Ansible Role: redhat_idm_groups

Manage Red Hat IdM groups.

## Using this role in an Ansible playbook

The following example shows how this role can be defined in a playbook with parameters passed to override default variables.

> [!INFORMATION]
> It is recommended that you use a Fully Qualified Collection Name (FQCN) when referencing your roles.

```yaml
---
- hosts: "{{ groups['idm_server'][0] }}"
  roles:
    - role: infra.soe.redhat_idm_groups
      redhat_idm_groups:
        - name: service-accounts
          description: Service Accounts
          users:
            - ansible
            - satellite
      tags: infra.soe.redhat_idm_groups
```

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
