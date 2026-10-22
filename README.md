# Ⓐ Ansible Collection: infra.soe

[![GitHub License](https://img.shields.io/github/license/snapp/ansible-collection-soe)](https://github.com/snapp/ansible-collection-soe/blob/main/LICENSE)[![Ansible Code of Conduct](https://img.shields.io/badge/Code%20of%20Conduct-Ansible-silver.svg)](https://docs.ansible.com/ansible/latest/community/code_of_conduct.html)
[![CI](https://github.com/snapp/ansible-collection-soe/actions/workflows/main.yml/badge.svg)](https://github.com/snapp/ansible-collection-soe/actions/workflows/main.yml)

## Description

`infra.soe` is a reference implementation of a Red Hat Standard Operating Environment (SOE). It demonstrates that the full Red Hat infrastructure stack can be bootstrapped from bare metal or minimal virtual machines using Red Hat Enterprise Linux (RHEL) and Ansible alone, with no OpenShift prerequisite: hardened image builds, zero-trust provisioning, Red Hat Identity Management (IdM) as the root of trust for certificates and DNS, Red Hat Satellite for content management, single sign-on through Red Hat build of Keycloak (RHBK), and Red Hat Ansible Automation Platform (AAP) as the control plane that then operates the estate.

The thesis behind it is that an organisation needs an automated RHEL foundation of Ansible, Satellite and IdM before it can adopt OpenShift well. This collection is that day-0 foundation layer. Day-1 and later automation (patching, compliance, application deployment) assumes it already exists, and OpenShift or MicroShift are future workloads that presuppose it rather than alternatives to it.

The patterns are production-grade at a minimum viable footprint: CIS-hardened images, end-to-end TLS with certificates issued by the IdM certificate authority through certmonger and ACME, vaulted credentials, idempotent configuration kept in git, Satellite content views that gate which package versions a host or an image build can see, and rootless Podman Quadlets for the self-hosted services. Where a single point of failure exists (one IdM server, one Satellite, one AAP node) it is a consequence of hardware footprint, not of the design; the roles accept replicas, capsules and multi-node installer inventories through the same inputs.

The collection is the product; a site repository is an instantiation of it with an inventory, `group_vars` and vaulted secrets, and the roles never assume a specific site. The reference site repository is [snapp/ansible-site-example](https://github.com/snapp/ansible-site-example).

> [!WARNING]
>
> This code is NOT supported by Red Hat and may not work as expected.
>
> Red Hat does not guarantee its correctness, reliability, or performance.
>
> Use at your own risk!

## Architecture

[`playbooks/site.yml`](https://github.com/snapp/ansible-collection-soe/blob/main/playbooks/site.yml) is the canonical build order. Each stage establishes something the next one depends on:

1. **Local users** (`local_users`) creates the local service accounts products insist on. Everything later is reached through this account until IdM takes over.
2. **RHEL registration** (`redhat_register`) registers every host for content, against Satellite once it exists or against Red Hat Subscription Management for the Satellite server itself. Every package installed afterwards comes from the repositories and content view this stage selects.
3. **Local storage** (`local_storage`) grows the root partition and lays out the LVM volumes, so that `/var`, `/var/log`, `/var/log/audit`, `/var/tmp` and the product volumes exist with their CIS mount options before anything is installed into them.
4. **Updates** (`redhat_update`) brings hosts to the promoted package versions, opt-in per group.
5. **IdM** (`redhat_identity_management`) installs the IdM server and replicas and configures the realm: users, groups, sudo and HBAC rules, password policies, DNS forwarders and zones, and the ACME service. From here on IdM is the certificate authority, the Kerberos realm and the DNS of the site.
6. **IdM client enrolment** (`redhat_idm_register`) enrols every other RHEL host, which is what later lets each host request certificates, enforce HBAC and sudo rules and resolve subordinate IDs for rootless containers through SSSD.
7. **Cockpit** (`redhat_cockpit`) requests a host certificate for the `HTTP/` principal of every enrolled host and optionally configures the web console on it. Every host now has an IdM-issued certificate to serve.
8. **Satellite** (`redhat_satellite`) installs Satellite Server, joins it to the IdM realm, puts the IdM certificate on it and configures organisations, content, lifecycle environments, content views, activation keys, host groups and provisioning from inventory. Content gating for hosts and image builds starts here.
9. **RHBK** (`redhat_keycloak`) installs Red Hat build of Keycloak with the IdM certificate, federates IdM users and groups, and configures the realms and the OpenID Connect clients that AAP, Satellite, the reverse proxy and NetBird authenticate against.
10. **Caddy** (`caddy`) deploys the rootless reverse proxy that terminates TLS for every self-hosted service, with certificates from the IdM ACME directory and optional oauth2-proxy sidecars that put RHBK single sign-on in front of applications that have none.
11. **NetBird** (`netbird`) deploys the self-hosted mesh VPN control plane behind Caddy, with RHBK as its identity provider, so remote peers reach the site over WireGuard.
12. **AAP** (`redhat_ansible_automation_platform`) installs containerized AAP with the IdM certificate and configures it from inventory. From here on this collection's playbooks would normally be run via job templates in AAP.
13. **Image Builder** (`redhat_image_builder`) installs RHEL image builder and points it at Satellite content views, so the next generation of hardened golden images is composed from promoted content.
14. **Application servers** (`application_server`) prepares the Podman hosts that carry further self-hosted workloads.
15. **Firewall** (`firewall`) applies the site-level firewalld definitions to every host, after all services have declared their ports.
16. **Reboot** (`reboot`) ends the run with a controlled reboot, only when asked for with `-e reboot=true`.

Cross-cutting concerns are factored into shared roles that the product roles compose: `redhat_idm_users`, `redhat_idm_groups`, `redhat_idm_hbac`, `redhat_idm_sudo`, `redhat_idm_acme` and `redhat_idm_service` for IdM objects and certificates, `firewall` for host filtering, `redhat_scap` for CIS remediation with per-product tailoring of justified exceptions, and `local_users` for local accounts.

## Included content

### Roles

Every role has a single `main` entry point, a `meta/argument_specs.yml` that is validated at role start, and a README with its variables, composition, tags, security considerations and operational notes.

#### Foundation

| Role | Purpose |
|---|---|
| [`bootstrap`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/bootstrap/README.md) | Load per-group vaulted bootstrap variable files from the site repository for the first run |
| [`local_users`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/local_users/README.md) | Manage local accounts with SSH keys and sudo on RHEL, Fedora and Windows |
| [`redhat_register`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_register/README.md) | Register a RHEL host with Satellite or Red Hat Subscription Management through the rhc system role |
| [`local_storage`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/local_storage/README.md) | Grow the root partition and lay out LVM volumes with the storage system role |
| [`redhat_update`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_update/README.md) | Update installed packages and optionally reboot |
| [`firewall`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/firewall/README.md) | firewalld services, policies, denied-packet logging and nftables quiet drops |
| [`reboot`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/reboot/README.md) | Reboot a host and wait for it to return |
| [`ping`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/ping/README.md) | Verify Ansible connectivity to RHEL, Fedora and Windows hosts |

#### Identity

| Role | Purpose |
|---|---|
| [`redhat_identity_management`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_identity_management/README.md) | Install and configure IdM servers and replicas as the site root of trust |
| [`redhat_idm_register`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_register/README.md) | Enrol a RHEL host as an IdM client |
| [`redhat_idm_users`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_users/README.md) | Manage IdM user accounts and subordinate ID ranges |
| [`redhat_idm_groups`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_groups/README.md) | Manage IdM groups and their membership |
| [`redhat_idm_hbac`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_hbac/README.md) | Manage IdM host-based access control rules |
| [`redhat_idm_sudo`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_sudo/README.md) | Manage IdM sudo rules |
| [`redhat_idm_acme`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_acme/README.md) | Enable or disable the IdM ACME certificate service |
| [`redhat_idm_service`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_idm_service/README.md) | Register an IdM service principal and manage its certmonger-issued TLS certificate |

#### Platform services

| Role | Purpose |
|---|---|
| [`redhat_satellite`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_satellite/README.md) | Install and configure Satellite Server with IdM integration, firewall rules and SCAP remediation |
| [`redhat_keycloak`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_keycloak/README.md) | Install RHBK with IdM integration and configure realms, clients and role mappings |
| [`redhat_ansible_automation_platform`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_ansible_automation_platform/README.md) | Install containerized AAP with IdM integration and configure it from inventory |
| [`redhat_image_builder`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_image_builder/README.md) | Install RHEL image builder and point its repositories at Satellite content views |
| [`redhat_cockpit`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_cockpit/README.md) | Configure the RHEL web console with an IdM-issued certificate |
| [`redhat_scap`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/redhat_scap/README.md) | Remediate a host against a SCAP profile with a tailoring of justified exceptions |

#### Edge and access

| Role | Purpose |
|---|---|
| [`caddy`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/caddy/README.md) | Rootless Podman Quadlet reverse proxy with optional RHBK forward-auth sidecars |
| [`netbird`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/netbird/README.md) | Self-hosted NetBird mesh VPN control plane as rootless Podman Quadlets |

#### Workload hosts

| Role | Purpose |
|---|---|
| [`application_server`](https://github.com/snapp/ansible-collection-soe/blob/main/roles/application_server/README.md) | Prepare a Podman application host and open its web ports |

### Playbooks

Every playbook selects its hosts with `hosts: "{{ target_hosts | default('<group>') }}"`, so `-e target_hosts=<pattern>` narrows a run without editing the playbook, and runs with `gather_facts: false` because the roles gather the facts they need.

| Playbook | Default target | Runs |
|---|---|---|
| `site` | the groups of the imported playbooks | the sixteen playbooks below in the build order described above |
| `local_users` | `all` | `infra.soe.local_users` |
| `redhat_register` | `redhat` | `infra.soe.redhat_register` |
| `local_storage` | `redhat:fedora` | `infra.soe.local_storage` |
| `redhat_update` | `redhat:!satellite` | `infra.soe.redhat_update` |
| `redhat_identity_management` | `idm_server:idm_replica` | `infra.soe.redhat_identity_management` |
| `redhat_idm_register` | `redhat:!identity_management` | `infra.soe.redhat_idm_register` |
| `redhat_idm_service` | `redhat` | `infra.soe.redhat_idm_service` standalone |
| `redhat_cockpit` | `redhat:!identity_management:!satellite` | `infra.soe.redhat_cockpit` |
| `redhat_satellite` | `satellite_server` | `infra.soe.redhat_satellite` |
| `redhat_keycloak` | `keycloak` | `infra.soe.redhat_keycloak` |
| `caddy` | `caddy` | `infra.soe.caddy` |
| `netbird` | `netbird` | `infra.soe.netbird` |
| `redhat_ansible_automation_platform` | `aap_installer` | `infra.soe.redhat_ansible_automation_platform` |
| `redhat_image_builder` | `image_builder` | `infra.soe.redhat_image_builder` |
| `application_server` | `application_server` | `infra.soe.application_server` |
| `firewall` | `redhat:fedora` | `infra.soe.firewall` |
| `reboot` | `all` | `infra.soe.reboot`, only when `reboot` is true |
| `redhat_scap` | `redhat` | `infra.soe.redhat_scap` standalone, for hosts no product role covers |
| `ping` | `all` | `infra.soe.ping` |

Each play carries tags (`redhat`, `fedora`, `microsoft` for the platform, plus a component tag such as `satellite` or `idm`), and the roles add lifecycle tags (`<component>_install`, `<component>_configure`, `idm_configure`, `firewall`, `scap`, ...) that are listed in each role README, so a job template can run `site` with `--tags` to select a stage.

## Shared inventory variables

Several roles read un-namespaced site-level variables, either as the fallback of a role-prefixed default or directly. This is the contract between the collection and a site repository; the roles never set these, and a site defines each once at the scope given below. Variables marked secret must come from Ansible Vault or an AAP credential injector. Every consuming role's README has a "Shared inventory variables consumed" section pointing back here, and the role READMEs explain the behaviour when a variable is unset.

| Variable | Type | Secret | Consumed by → role variable fed | Usual scope |
|---|---|---|---|---|
| `no_log_enabled` | bool | – | `caddy`, `netbird`, `redhat_identity_management`, `redhat_idm_users`, `redhat_idm_groups`, `redhat_idm_hbac`, `redhat_idm_sudo`, `redhat_idm_service`, `redhat_keycloak`, `redhat_satellite` (read directly; unset behaves as `true`) | `all` |
| `idm_admin_password` | str | yes | `caddy`, `redhat_ansible_automation_platform`, `redhat_cockpit`, `redhat_image_builder`, `redhat_keycloak`, `redhat_satellite` → `<role>_idm_admin_password`; `redhat_idm_register` → `redhat_idm_register_admin_password`; `redhat_idm_users` (Kerberos login, read directly) | `redhat` |
| `idm_admin_user` | str | – | `redhat_idm_register` → `redhat_idm_register_admin_user` | `redhat` |
| `idm_domain` | str | – | `redhat_identity_management`, `redhat_idm_service`, `redhat_satellite` (read directly); `redhat_idm_register` → `redhat_idm_register_domain` | `all` |
| `idm_client_subid`, `idm_client_mkhomedir`, `idm_client_allow_repair`, `idm_client_force`, `idm_client_force_join`, `idm_client_all_ip_addresses` | bool | – | `redhat_idm_register` → the matching `redhat_idm_register_*` | `redhat` |
| `idm_default_group`, `idm_default_shell`, `idm_default_email_domain`, `idm_default_home_directory`, `idm_default_search_size_limit`, `idm_enable_sid`, `idm_add_sids`, `idm_service_account_max_life`, `idm_admin_account_max_life`, `idm_domain_external_ip_address`, `idm_global_forwarders`, `idm_forwarder_zones`, `idm_dns_zones`, `idm_ip`, `ipaserver_reverse_zones`, `bootstrap_user` | mixed | – | `redhat_identity_management` (read directly; no role-prefixed counterpart) | `identity_management` |
| `idm_primary_server` | str | – | `redhat_idm_acme` (read directly) | `identity_management` |
| `redhat_idm_users`, `redhat_idm_groups`, `redhat_idm_sudo_rules`, `redhat_idm_hbac_rules`, `redhat_idm_acme_enabled` | list, bool | passwords inside | the `redhat_idm_*` leaf roles when composed by `redhat_identity_management` (site-wide accounts and rules) | `identity_management` |
| `rhc_server` and the other `rhc_*` inputs of `redhat.rhel_system_roles.rhc` | dict | – | `redhat_register` (`rhc_server.hostname` selects Satellite or RHSM registration) | `redhat`, overridden per group |
| `satellite_hostname`, `satellite_location`, `satellite_host_group` | str | – | `redhat_register` (read directly) | `redhat`, overridden per group |
| `satellite_server_url`, `satellite_username`, `satellite_password`, `satellite_validate_certs`, `satellite_organization` | str, bool | username and password | `redhat_register`, `redhat_satellite` (read directly; also the connection inputs of every `redhat.satellite` role) | `redhat` |
| `satellite_manifest_import`, `satellite_host_collections`, `satellite_realms` | bool, list | – | `redhat_satellite` (read directly) | `satellite_server` |
| `satellite_baseurl`, `satellite_organization` | str | – | `redhat_image_builder` → `redhat_image_builder_satellite.baseurl` and `.organization` | `redhat` |
| `keycloak_base_url`, `keycloak_primary_realm` | str | – | `caddy` → `caddy_keycloak_base_url`, `caddy_keycloak_primary_realm`; `redhat_keycloak` → `redhat_keycloak_base_url`, `redhat_keycloak_primary_realm` | `all` |
| `rhbk_bootstrap_admin_user`, `rhbk_bootstrap_admin_password` | str | password | `redhat_keycloak` → `redhat_keycloak_admin_user`, `redhat_keycloak_admin_password` | `keycloak` |
| `rhbk_url`, `rhbk_clients`, `rhbk_client_rolemappings`, `rhbk_user_rolemappings` | str, list | client secrets inside | `redhat_keycloak` (read directly; `rhbk_*` is also the input namespace of `redhat.rhbk`) | `keycloak` |
| `aap_routable_hostname`, `aap_gateway_admin_password`, `aap_gateway_validate_certs`, `aap_manifest_import`, `aap_controller_license` | mixed | admin password | `redhat_ansible_automation_platform` (read directly) | `aap_installer` host |
| `aap_service_user`, `aap_setup_*`, `aap_backup_dest` | mixed | offline token and the passwords inside `aap_setup_prep_inv_vars` | `redhat_ansible_automation_platform` (pass-through to `infra.aap_utilities` and `infra.aap_configuration`) | `aap_installer` host |
| `podman_pull_image` | bool | – | `caddy`, `netbird` (read directly; `redhat.rhel_system_roles.podman` input) | `application_server` |
| `cockpit_port` and the other `cockpit_*` inputs of `redhat.rhel_system_roles.cockpit` | int | – | `redhat_cockpit` (read directly) | `redhat` |
| `redhat_scap_profile`, `redhat_scap_report_dir` | str | – | `redhat_scap`; the product roles mirror it as `<role>_scap_profile: "{{ redhat_scap_profile }}"` | `redhat` |
| `bootstrap`, `reboot` | bool | – | gates of the `bootstrap` role and the `reboot` playbook | extra variables |

Naming is not uniform across these variables (`keycloak_*` and `rhbk_*` both refer to RHBK; `aap_*` are upstream names; the `idm_*` inputs of `redhat_identity_management` have no role prefix). Renaming them would break every site repository, so they are documented as they are.

## Requirements

Collection dependencies, declared in [`galaxy.yml`](https://github.com/snapp/ansible-collection-soe/blob/main/galaxy.yml) and installed automatically with the collection:

| Collection | Used for |
|---|---|
| [`redhat.rhel_idm`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/rhel_idm/) | IdM server, replica and client roles; the `ipa*` modules the identity roles use |
| [`redhat.rhel_system_roles`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/rhel_system_roles/) | `firewall`, `storage`, `rhc`, `cockpit` and `podman` roles, `sefcontext` and `seport` modules |
| [`redhat.satellite`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite/) | Satellite configuration roles and the `host`, `host_collection` and `realm` modules |
| [`redhat.satellite_operations`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite_operations/) | the Satellite `installer` role |
| [`redhat.rhbk`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/rhbk/) | RHBK installation, realm configuration and the `rhbk_client` module |
| [`infra.aap_utilities`](https://console.redhat.com/ansible/automation-hub/repo/validated/infra/aap_utilities/) | AAP containerized installer download, preparation and installation |
| [`infra.aap_configuration`](https://console.redhat.com/ansible/automation-hub/repo/validated/infra/aap_configuration/) | AAP manifest import and declarative configuration dispatch |
| [`community.general`](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) | `rpm_ostree_pkg`, `sefcontext` and the Keycloak role mapping modules |
| [`ansible.windows`](https://console.redhat.com/ansible/automation-hub/repo/published/ansible/windows/), [`community.windows`](https://docs.ansible.com/ansible/latest/collections/community/windows/index.html) | Windows local users and connectivity checks |

The `caddy` and `netbird` roles also use `ansible.posix.seboolean`; [`ansible.posix`](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html) is not yet declared as a dependency and must be installed alongside. The `redhat.*` collections come from Red Hat Automation Hub (certified content) and the `infra.aap_*` collections from its validated content; both need an Automation Hub token (see Installation). No Python packages beyond those required by these collections are needed on the control node.

Managed nodes are RHEL 9 and 10. `application_server`, `firewall`, `local_storage` and `local_users` also run on Fedora, and `local_users` and `ping` on Windows. The control node must reach the RHBK API and trust the IdM certificate authority, because `redhat_keycloak` configures realms and clients from the control node.

## Installation

### Automation Hub token

The dependencies published on Red Hat Automation Hub require authentication. Create an offline token at https://console.redhat.com/ansible/automation-hub/token and configure the servers in `ansible.cfg`:

```ini
[galaxy]
server_list = redhat_automation_hub_certified, redhat_automation_hub_validated, community

[galaxy_server.community]
url=https://galaxy.ansible.com/

[galaxy_server.redhat_automation_hub_certified]
url=https://console.redhat.com/api/automation-hub/content/published/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<your offline token>

[galaxy_server.redhat_automation_hub_validated]
url=https://console.redhat.com/api/automation-hub/content/validated/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<your offline token>
```

### Installing the collection

From Ansible Galaxy:

```console
ansible-galaxy collection install infra.soe
```

From the repository, for a development version:

```console
ansible-galaxy collection install git+https://github.com/snapp/ansible-collection-soe.git,main
```

Or build it locally after cloning and checking out a tag:

```console
ansible-galaxy collection build .
ansible-galaxy collection install infra-soe-*.tar.gz
```

### Installing from a requirements file

```yaml
---
collections:
  - name: infra.soe
  - name: ansible.posix
```

```console
ansible-galaxy collection install -r requirements.yml
```

### Installing a specific version and upgrading

```console
ansible-galaxy collection install infra.soe:==0.27.0
ansible-galaxy collection install infra.soe --upgrade
```

A collection installed from Galaxy is not upgraded with the `ansible` package. See [Using collections](https://docs.ansible.com/ansible/latest/collections_guide/index.html) for details.

## Use cases

**Stand up a new site from minimal RHEL virtual machines.** Create a site repository with the inventory groups the playbooks target (`redhat`, `idm_server`, `satellite_server`, `keycloak`, `caddy`, `netbird`, `aap_installer`, `image_builder`, `application_server`), the shared inventory variables above and vaulted bootstrap files, then run `infra.soe.site` with `-e bootstrap=true` for the first pass. The run ends with IdM issuing certificates, Satellite serving content, RHBK providing single sign-on and AAP ready to take over the playbooks as job templates.

**Add a Satellite-managed, CIS-hardened host.** Register the host with an activation key and host group (`redhat_register`), apply its storage layout (`local_storage`), enrol it in IdM (`redhat_idm_register`), give it an IdM certificate and the web console (`redhat_cockpit`), and remediate it against the CIS profile with site-specific exceptions (`redhat_scap`), each exception carrying its justification and review date.

**Put single sign-on in front of a self-hosted application.** Define a confidential RHBK client with a `groups` and an audience mapper (`redhat_keycloak`), add a `caddy_oauth2_proxies` entry and a site block that uses `forward_auth` (`caddy`), and the application is reachable only by members of the allowed IdM group, without the application knowing anything about OpenID Connect.

**Issue an IdM certificate to any service.** Run `redhat_idm_service` for a host with its service account, request identifier and aliases, and certmonger keeps an IdM-issued certificate, key and CA chain under `/etc/pki/<id>/` for the service to use and renews it automatically.

**Run a patch cycle gated by content views.** Publish and promote a content view in Satellite (`redhat_satellite` inputs), then run `redhat_update` with `redhat_update_enabled=true` and `redhat_update_reboot_enabled=true` against the lifecycle environment's hosts; only the promoted package versions are visible to them.

## Testing

Continuous integration ([`.github/workflows/main.yml`](https://github.com/snapp/ansible-collection-soe/blob/main/.github/workflows/main.yml)) builds and installs the collection with its dependencies, runs the pre-commit hook set (gitleaks, ansible-lint with the production profile in strict mode, yamllint, the role documentation drift check and the generic file checks), and runs `antsibull-docs lint-collection-docs` against the role argument specs. The same checks run locally with `pre-commit run --all-files` and `antsibull-docs lint-collection-docs . --plugin-docs --skip-rstcheck`.

The roles are exercised against the reference site, with RHEL 9 and 10 managed nodes and a single node per product. There is no Molecule or integration test suite yet; adding one is on the roadmap. Known limitations are documented in each role README under "Operational notes".

## Contributing

Contributions are welcome. See [`CONTRIBUTING.md`](https://github.com/snapp/ansible-collection-soe/blob/main/CONTRIBUTING.md) for the development process, the documentation conventions (also in [`AGENTS.md`](https://github.com/snapp/ansible-collection-soe/blob/main/AGENTS.md)) and the changelog fragment workflow. This project follows the [Ansible Community Code of Conduct](https://docs.ansible.com/ansible/latest/community/code_of_conduct.html).

## Support

This collection is community content and is not supported by Red Hat. Report problems and requests through the GitHub issue tracker at https://github.com/snapp/ansible-collection-soe/issues. Support for the Red Hat products the collection deploys is a matter of the subscriptions covering them, not of this collection.

## Release notes and roadmap

Release notes are in [`CHANGELOG.rst`](https://github.com/snapp/ansible-collection-soe/blob/main/CHANGELOG.rst), generated from fragments with antsibull-changelog. The collection is pre-1.0 and its interface (variable names, tags) may still change between minor versions; the changelog's "Breaking Changes / Porting Guide" section lists every such change. Planned before 1.0.0: a Molecule test suite, consistent variable and tag naming, and automated registration of RHBK as the NetBird identity provider. OpenShift and MicroShift are future workloads on top of this foundation.

## Related information

- [snapp/ansible-site-example](https://github.com/snapp/ansible-site-example), the reference site repository with inventory, `group_vars` and bootstrap layout
- [Red Hat Ansible Automation Platform documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/)
- [Red Hat Satellite documentation](https://docs.redhat.com/en/documentation/red_hat_satellite/)
- [Red Hat Identity Management documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/installing_identity_management/index)
- [Red Hat build of Keycloak documentation](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/)
- [Red Hat Community of Practice good practices for Ansible](https://redhat-cop.github.io/automation-good-practices/), which this collection's conventions follow

## License

GNU General Public License v3.0 or later. See [LICENSE](https://github.com/snapp/ansible-collection-soe/blob/main/LICENSE) for the full text.
