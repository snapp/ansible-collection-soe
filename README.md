# Ⓐ Ansible Collection: infra.soe


[![GitHub License](https://img.shields.io/github/license/snapp/ansible-collection-soe)](https://github.com/snapp/ansible-collection-soe/blob/main/LICENSE)[![Ansible Code of Conduct](https://img.shields.io/badge/Code%20of%20Conduct-Ansible-silver.svg)](https://docs.ansible.com/ansible/latest/community/code_of_conduct.html)
[![CI](https://github.com/snapp/ansible-collection-soe/actions/workflows/main.yml/badge.svg)](https://github.com/snapp/ansible-collection-soe/actions/workflows/main.yml)

Standard Operating Environments (SOE)

> [!WARNING]
>
> This code is NOT supported by Red Hat and may not work as expected.
>
> Red Hat does not guarantee its correctness, reliability, or performance.
>
> Use at your own risk!

## Using this collection

### Build and install locally

Clone the repository, checkout the tag you want to build, or pick the main branch for the development version; then:

    ansible-galaxy collection build .
    ansible-galaxy collection install infra-soe-*.tar.gz

### Installing the Collection from Ansible Galaxy

Before using this collection, you need to install it with the Ansible Galaxy command-line tool:

```console
ansible-galaxy collection install infra.soe
```

You can also include it in a `requirements.yml` file and install it with `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: infra.soe
```

Note that if you install the collection from Ansible Galaxy, it will not be upgraded automatically when you upgrade the `ansible` package. To upgrade the collection to the latest available version, run the following command:

```console
ansible-galaxy collection install infra.soe --upgrade
```

You can also install a specific version of the collection, for example, if you need to downgrade when something is broken in the latest version (please report an issue in this repository). Use the following syntax to install version `1.0.0`:

```console
ansible-galaxy collection install infra.soe:==1.0.0
```

See [Ansible Using collections](https://docs.ansible.com/ansible/devel/user_guide/collections_using.html) for more details.

## Licensing

GNU General Public License v3.0 or later

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.
