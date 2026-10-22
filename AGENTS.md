# Conventions for infra.soe

This file records the documentation and interface conventions of the `infra.soe` collection, for people and for coding agents. Each rule names its source: **[CERT]** the Red Hat partner certification README requirements (https://docs.ansible.com/projects/partner-certification-requirements/readmes/), **[TMPL]** the certified collection README template (https://github.com/ansible-collections/partner-certification-requirements/blob/main/docs/readme-template.md), **[CoP]** the Red Hat Community of Practice good practices for Ansible (https://redhat-cop.github.io/automation-good-practices/), **[CORE]** the ansible-core role argument validation documentation (https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-argument-validation), and **[REPO]** a decision made in this repository.

## What this repository is

- `infra.soe` is a Red Hat Standard Operating Environment (SOE) **reference implementation**: the full Red Hat infrastructure stack bootstrapped from minimal RHEL with Ansible alone, as the day-0 foundation that later automation and OpenShift presuppose. Write every document so a reader understands why a role exists, not only what it does. [REPO]
- The collection is the product; a site repository (inventory, `group_vars`, vaulted secrets) is an instantiation. Roles never assume a specific site. The reference site is https://github.com/snapp/ansible-site-example. [REPO] [CoP 5.2]
- Production-grade patterns at minimum viable footprint. Where a single point of failure exists, say that it comes from hardware footprint, not design. [REPO]
- This repository is public and should not contain any private or sensitive information. [REPO]

## Terminology

- Never "lab", "homelab", "demo" or "test environment". It is a reference implementation. This applies to comments in `defaults/` and to the justification text in `vars/scap/*.yml`. [REPO]
- Hierarchy: estate → environment → site. Sites are inventory groups, never git branches. [REPO]
- Site identifiers are IATA airport codes plus an index (`mtj1`). Host names follow `<role><n>[-<iface>].<site>.<env>.<parent>` with `example.org` (RFC 2606) as the parent and `prod` or `test` as the environment token, for example `idm1.mtj1.prod.example.org`, `sat1.mtj1.prod.example.org`, `kc1.mtj1.prod.example.org`, `aap1.mtj1.prod.example.org`, `ib1.mtj1.prod.example.org`, `app1.mtj1.prod.example.org`. Service aliases are `<service>.<site>.<env>.<parent>`, for example `satellite.mtj1.prod.example.org`. Network zones are `10.<site-index>.<zone-index>.0/24`, for example `10.1.10.0/24`. Never `.int`, `.mgt`, `.lab` or `dc1`-style names in examples. [REPO]
- "Zone" is overloaded; always qualify it: firewalld zone, DNS zone, network zone. [REPO]
- Product names are spelled out on first use in every document: Red Hat Enterprise Linux (RHEL), Red Hat Identity Management (IdM), Red Hat Satellite, Red Hat build of Keycloak (RHBK), Red Hat Ansible Automation Platform (AAP), RHEL image builder. [REPO]
- Keep the "NOT supported by Red Hat" warning in the collection README; issues go to the GitHub tracker. [CERT] [REPO]

## Collection README

- Section order follows the certified template [TMPL]: Description (with the support warning), Architecture (the `site.yml` build order as a numbered narrative, each stage explaining what it establishes for the next), Included content (roles by family and playbooks), Shared inventory variables, Requirements, Installation, Use cases, Testing, Contributing, Support, Release notes and roadmap, Related information, License.
- Requirements list collection dependencies with what each is used for and the supported platforms. Do not list `ansible-core`; its floor is declared in `meta/runtime.yml`. [CERT]
- Installation covers Automation Hub (with the token configuration the `redhat.*` and `infra.aap_*` dependencies need), Galaxy, git and a `requirements.yml` example. Never `pip install ansible` or `pip install ansible-core`. [CERT]
- Three to five concrete use cases. [TMPL]

## Links

- Every link in every README is a full URL in Markdown syntax, never a relative path, because READMEs are rendered on Galaxy and Automation Hub outside the repository. Repository files are linked as `https://github.com/snapp/ansible-collection-soe/blob/main/<path>`. [CERT]
- `tools/render_role_docs.py --check` fails on relative links. [REPO]

## Role README

Heading `# Ⓐ Ansible Role: <role_name>`, then these sections in this order, omitting only sections that are truly empty [CoP 4.1.17] [REPO]:

1. **Summary**: one or two paragraphs. What it configures, where it sits in the `site.yml` build order, what must already exist, what depends on it, and why it exists in the reference implementation.
2. The FQCN note, as a GitHub alert: `> [!NOTE]`. Valid alert types are `NOTE`, `TIP`, `IMPORTANT`, `WARNING`, `CAUTION`; `[!INFORMATION]` is invalid and the check rejects it.
3. **Requirements**: managed node platforms, collections used directly (FQCN and full URL), prerequisites (IdM enrolment, service account with linger, manifest, ...), required inventory groups.
4. **Role variables**: the generated section between `<!-- BEGIN ROLE VARIABLES -->` and `<!-- END ROLE VARIABLES -->`, with a sentence about secrets above it. Never edit the generated text by hand.
5. **Shared inventory variables consumed**: every un-namespaced variable the role reads, the role variable it feeds, and a link to the collection README section.
6. **Dependencies and composition**: roles imported or included, meta dependencies, and which variables are passed through to each.
7. **Tags**: every tag the role responds to, what it selects, and which are shared across roles. Document tags as they are.
8. **Example playbook**: matching `playbooks/<role>.yml` (`hosts: "{{ target_hosts | default('<group>') }}"`, `gather_facts: false`, FQCN). Use only variables that exist.
9. **Security considerations**: `no_log` handling, files written with sensitive modes, ports opened, services exposed, privilege use, delegation.
10. **Operational notes**: idempotency caveats, ordering, known gotchas. Keep hard-won notes (the `caddy` and `netbird` READMEs) when restructuring.
11. **License** (full URL to the LICENSE file) and **Author**.

Describe behaviour the tasks actually have. When a default is defined but no task reads it, say so instead of describing what it would do.

## Argument specs (`meta/argument_specs.yml`)

- One entry point per tasks file that is used as an entry point. Today every role has only `main`; no playbook or role uses `tasks_from`. [CORE]
- Every non-internal key in `defaults/` (including split files under `defaults/main/`) is an option, and the option's `default` equals the `defaults/` value literally, Jinja included, because the runtime default always comes from `defaults/`; the spec default is documentation. [CORE] [CoP 4.1.20]
- Inputs the tasks read without a default (`| default(...)`, `is defined` gates, pass-through names) are specified without a `default` and with a description of the behaviour when unset. Variables that only an upstream role reads are not specified; the README links the upstream documentation instead. [REPO]
- Internal variables (`__<role>_*`, `_<role>_*`), registered variables and `vars/` constants are never specified. The drift check skips `_`-prefixed `defaults/` keys. [CoP 4.1.4]
- `short_description` is one line without a trailing period and is identical to `galaxy_info.description` in `meta/main.yml`. `description` is a list of full sentences. Never write "Defaults to X" in a description; the default is already shown. [CORE]
- Secrets: `no_log: true` plus the word "Secret" at the start of the description. ansible-core honours `no_log` in role specs (values are masked in validation errors), ansible-lint and antsibull-docs accept it, `ansible-doc` does not render it. [REPO]
- `required: true` only for an input the role cannot work without, that has no default, and that the role either asserts itself or dereferences before any gate. Current set: `caddy_domains`; `netbird_relay_secret`, `netbird_datastore_encryption_key`, `netbird_session_cookie_key`, `netbird_idp_client_secret`; `redhat_identity_management_admin_password`; `redhat_idm_users_admin_password`, `redhat_idm_groups_admin_password`, `redhat_idm_hbac_admin_password`, `redhat_idm_sudo_admin_password`, `redhat_idm_service_admin_password`; `bootstrap_directory`; `rhc_server`. Shared un-namespaced variables are never required. [REPO]
- `choices` only where the tasks enforce a closed set. [REPO]
- Types follow the value after templating, with these constraints learned from the validators:
  - An option whose `defaults/` value is `{{ ... | default(omit) }}` is `type: str`; the omit placeholder is a string and a `list` or `dict` type rejects it at runtime. An option whose default is either a structure or omit is `type: raw`. [REPO]
  - antsibull-docs validates a spec `default` against its declared type, so an option whose `defaults/` value is a Jinja expression rendering to a boolean, an integer, a list or a dict is `type: raw`, with the effective type stated in the description (for example "Boolean."). YAML numbers such as `2.7` are also `raw`. [REPO]
  - Nested `options` reject undeclared keys at runtime. Use them only where the role itself asserts or renders every key (`firewall_policies`, `firewall_quiet_drops`, `caddy_oauth2_proxies`, `redhat_scap_tailoring_exceptions`, `redhat_image_builder_satellite`). Describe the item keys in prose everywhere else, in particular for the IdM user, group, HBAC and sudo lists and anything a site authors freely. [REPO]
  - Structures passed straight through to another collection's role are `type: dict` or `list` with a link to the upstream documentation; their schema is not re-specified. [REPO]
  - File system paths are `type: path`. [REPO]
- Plain YAML scalars in descriptions must not contain a colon followed by a space; quote the string or rephrase. [REPO]
- After changing a spec, run `ansible-doc -t role infra.soe.<role>` against an installed build and confirm it prints `ENTRY POINT: main`; `ansible-doc` exits 0 even for a role without a spec. [REPO]

## `meta/main.yml`

- Same layout as `roles/redhat_idm_service/meta/main.yml`: `role_name`, `description` (identical to the spec `short_description`), `author`, `license: GPL-3.0-or-later`, `min_ansible_version: ">=2.15.0"` (the floor in `meta/runtime.yml`), `platforms`, `galaxy_tags`. Existing `dependencies:` blocks are kept as they are. [REPO]
- Platforms only with evidence in code: EL 9 and 10 for every role (`vars/RedHat_9.yml`, SCAP tailoring written against RHEL 9 and 10, `aap_setup_rhel_version: 10`); Fedora for `application_server`, `firewall`, `local_storage` and `local_users`, whose playbooks target `redhat:fedora`; Windows for `local_users` and `ping`, which have Windows task files. `reboot` has an empty Windows task file and claims EL only. EL 8 is not claimed. [REPO]
- `galaxy_tags` are lowercase alphanumeric words that describe the role; never a placeholder such as `tools`. [REPO]

## Shared inventory variables

- Roles may fall back to un-namespaced site variables (`idm_admin_password`, `satellite_*`, `keycloak_*`, `no_log_enabled`, ...). The complete contract is the "Shared inventory variables" table in the collection README; each consuming role README points to it. Prefer the pattern `<role>_x: "{{ shared_x | default(...) }}"` in `defaults/` over reading the shared name in tasks. [CoP 5.2]
- Do not rename variables or tags in a documentation pass; renames break every site repository and belong in a findings report with a migration plan. [REPO]

## Naming (recorded, enforced only for new code)

- Role variables carry the role name as prefix; internal variables carry `__<role>_`; tags carry the role name. [CoP 4.1.4]
- `snake_case` everywhere; positive boolean names (`enabled`, not `disabled`). [CoP 9.1, 9.2]
- Tags are used to switch roles or complete purposes and are documented in the README. [CoP 6.3]

## Single source of truth tooling

- `tools/render_role_docs.py` renders the variables section of every role README from `meta/argument_specs.yml` and, with `--check`, fails on drift between README, spec and `defaults/` as well as on the rules above. The `role-docs` pre-commit hook runs the check. `tools/` is build-ignored. [REPO]
- Commonly set variables (required, no default, or secret) and advanced variables (with a default) are split by the renderer; structured defaults longer than one line link to the defaults file. [REPO]

## Changelog

- One antsibull-changelog fragment per change in `changelogs/fragments/`, sections as in `changelogs/config.yaml`, entries written as `<role or area> - Sentence.` in ReStructuredText. New argument specs are `minor_changes`; a spec that would reject a previously working inventory is a `breaking_changes` entry. [REPO]
- Versions are decided by the maintainer; a documentation change does not bump `galaxy.yml`. [REPO]

## Git

- Commit subjects are `<area>: <imperative summary>` (`docs: ...`, `tools: ...`, `redhat_satellite: ...`); the body explains why. Commits carry a `Signed-off-by` trailer. No force pushes, no history rewrites on shared branches. [REPO]

## Verification

Run from the repository root, with the Automation Hub token configured:

```console
ansible-galaxy collection build . --force
ansible-galaxy collection install infra-soe-*.tar.gz --force
for r in roles/*/; do ansible-doc -t role "infra.soe.$(basename "$r")" | grep -q '^ENTRY POINT: main' || echo "NO SPEC $r"; done
antsibull-docs lint-collection-docs . --plugin-docs --skip-rstcheck
ansible-lint
pre-commit run --all-files
python3 tools/render_role_docs.py --check
tar tzf infra-soe-*.tar.gz | grep -E '^(AGENTS\.md|CLAUDE\.md|tools/|\.ansible/)' && echo "files leaked into the artifact"
```

All of these must pass before a change is considered done. If a check fails for a reason outside the change (lint debt, dependency installation), report the exact error rather than suppressing the rule.
