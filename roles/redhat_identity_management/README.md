# Ⓐ Ansible Role: redhat_identity_management

Configure Red Hat Identity Management (IdM) — global defaults, users, groups, sudo
rules, HBAC rules, ACME, DNS, password policies, and bootstrap-user cleanup.

This role composes the standalone `redhat_idm_*` roles rather than duplicating their
logic:

- `tasks/network.yml` — gathers facts across `idm_server`/`idm_replica` peers and
  stitches `/etc/hosts` entries so IdM install/replica plays can resolve each other.
  Invoked separately, via `tasks_from: network`, from an earlier play in
  `playbooks/redhat_identity_management.yml` that runs before IdM is installed — this
  role's default entrypoint (`tasks/main.yml`) assumes IdM is already installed.
- `tasks/configure.yml` — kinit as IdM admin, `ipaconfig` global defaults, then
  `import_role` for `infra.soe.redhat_idm_{acme,users,groups,sudo,hbac}` (each keeping
  its own tag for selective `--tags` runs), password policies, and DNS
  forwarders/zones/records. Wrapped in `run_once: true` since it targets a single IdM
  server.
- `tasks/cleanup.yml` — removes the local bootstrap user once IdM-based login is
  confirmed working.

## Using this role in an Ansible playbook

> [!INFORMATION]
> It is recommended that you use a Fully Qualified Collection Name (FQCN) when referencing your roles.

```yaml
---
- hosts: "{{ groups['idm_server'][0] }}"
  roles:
    - role: infra.soe.redhat_identity_management
```

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
