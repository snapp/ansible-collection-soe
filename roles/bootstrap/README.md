# Ⓐ Ansible Role: bootstrap

Discover Host Group bootstrap variable files

## Using this role in an Ansible playbook

The following example shows how this role can be defined in a playbook with parameters passed to override default variables.

> [!INFORMATION]
> It is recommended that you use a Fully Qualified Collection Name (FQCN) when referencing your roles.

```yaml
---
- name: Load Host Group bootstrap variable files
  hosts: all
  gather_facts: false
  become: false
  tags: always
  vars:
    bootstrap_directory: "{{ playbook_dir }}/bootstrap"
  tasks:
    - name: Check for bootstrap mode
      ansible.builtin.include_role:
        name: infra.soe.bootstrap
      when:
        - bootstrap | default(false) | bool
```

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
