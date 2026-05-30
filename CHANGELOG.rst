=======================
infra.soe Release Notes
=======================

.. contents:: Topics

v0.23.0
=======

Minor Changes
-------------

- redhat_keycloak - Added playbook to install Red Hat Build of Keycloak (RHBK)

v0.22.0
=======

Minor Changes
-------------

- local_users - Add support for disabling selinux home equivalent for service accounts whose home dir is an app directory

v0.21.0
=======

Minor Changes
-------------

- redhat_identity_management - Remove local bootstrap user after IdM initialization
- redhat_idm_register - Add idm_client_all_ip_addresses
- redhat_idm_service - Fix IdM service CNAME
- redhat_idm_users - Ensure kerberos init before subid range enablement

v0.20.0
=======

Minor Changes
-------------

- image_builder - Added playbook to install Red Hat Image Builder

v0.19.0
=======

Minor Changes
-------------

- reboot - relocate task logic to standalone cross-platform role

v0.18.0
=======

Minor Changes
-------------

- redhat_cockpit - Add TLS certification via IdM service
- redhat_cockpit - Allow websm_port_t on port 443 when necessary
- redhat_cockpit - Ensure cockpit systemd units are not masked

v0.17.0
=======

Minor Changes
-------------

- scap_remediation - Added playbook for Security Content Automation Protocol (SCAP) remediation

v0.16.0
=======

Minor Changes
-------------

- bootstrap - Added role to discover host group bootstrap variable files

v0.15.0
=======

Minor Changes
-------------

- local_storage - Added playbook to configure local storage

v0.14.0
=======

Minor Changes
-------------

- redhat_ansible_automation_platform - Added playbook to install Red Hat Ansible Automation Platform (AAP)

v0.13.0
=======

Minor Changes
-------------

- redhat_satellite - Add IdM Service support to redhat_satellite

v0.12.0
=======

Minor Changes
-------------

- redhat_idm_service - Added playbook to configure Red Hat Identity Management (IdM) service integration

v0.11.0
=======

Minor Changes
-------------

- local_users - Added playbook to manage local user accounts

v0.10.0
=======

Minor Changes
-------------

- ping - Added playbook to update verify Ansible connectivity

v0.9.0
======

Minor Changes
-------------

- reboot - Added playbook to reboot hosts

v0.8.0
======

Minor Changes
-------------

- redhat_idm_register - Added playbook to perform Red Hat Identity Management (IdM) Client Registration

v0.7.0
======

Minor Changes
-------------

- redhat_identity_management - Added playbook to install Red Hat Identity Management (IdM)

v0.6.0
======

Minor Changes
-------------

- redhat_satellite - Added playbook to install Red Hat Satellite Server

v0.5.0
======

Minor Changes
-------------

- redhat_cockpit - Added playbook to configure Red Hat Enterprise Linux (RHEL) Cockpit Web Console

v0.4.0
======

Minor Changes
-------------

- firewall - Added playbook to configure Red Hat Enterprise Linux (RHEL) firewall

v0.3.0
======

Minor Changes
-------------

- redhat_update - Added playbook to update Red Hat Enterprise Linux (RHEL) Systems

v0.2.0
======

Minor Changes
-------------

- redhat_register - Added playbook to handle Red Hat Subscription Management (RHSM) registration.
