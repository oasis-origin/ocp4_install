ocp4_install
============

This repository contains reference plays for creating and destroying OpenShift Container Platform
4.x using OASIS roles. They should work on their own, while also being easily customizeable.

The plays are contained in the reference playbook, in [playbooks/site.yml](playbooks/site.yml).

Requirements/Prerequisites
==========================

Ansible 2.9 or higher

Valid Red Hat Subscriptions, included access to documentation at access.redhat.com. Note that
a subscribed system is not required to run the installer, but valid subscribtions are needed
to generate the pull secrets (handled by the `ocp_pull_secrets` role),  needed to pull down
official product images role below.

General familiarity with manually installing OCP 4.x on the platform of choice, including
generating credentials and assigning required permissions, is assumed.

Dependencies
============

OASIS Roles:

- [index_href](https://galaxy.ansible.com/oasis_roles/index_href), used to generate
  direct URLs to openshift archives given an index URL. No special configuration is
  required for this role.
- [ocp_pull_secrets](https://galaxy.ansible.com/oasis_roles/ocp_pull_secrets), used
  to generate the pull secrets used by the deployed OCP cluster, documented below.
- [ocp_install](https://galaxy.ansible.com/oasis_roles/ocp_install), which does the
  work of downloading, installing, configuring, and then running `openshift-installer`.
  Basic deployments are implemented in this repository; for more advanced deployments,
  complete control of the OCP 4 `install-config.yaml` is offered by this role.

Groups and Group Vars
=====================

The `site.yml` playbooks supports the following groups:

- `all`
- `aws`
- `azure`
- `gcp`

The `all` group provides variable values common to all OpenShift 4 providers.
The provider-specific groups (e.g. `aws`, `azure`, etc.) provide variable values
specific to that provider, such as credentials and configuration paths.

Example `group_vars` have been provided in the `group_vars` directory, which
should be sufficient to create an OpenShift 4.x cluster using the `playbooks/site.yml`
reference playbook. If altering this playbook to suit your needs, note that altering
the `group_vars` will almost certainly be necessary as well.

[all](group_vars/all.example)
---

The following variables are used in the `group_vars/all.example` to set the
corresponding role variables needed to deploy an OpenShift 4.x cluster
on any platform.

* `cluster_name` - Name of the cluster, used for the value of `ocp_install_cluster_name`
  when invoking the `ocp_install` role.
* `base_domain` - Name of base DNS domain, used for the value of `ocp_install_base_domain`
  when invoking the `ocp_install` role.
* `ssh_pubkey_file` - Path to a file containing an SSH public key, used to provide the value
  for `ocp_install_ssh_pubkey` via a `file` lookup.
* `ocp_installers_index_url` - URL of the openshift file index containing installers here,
  such as https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/
* `ocp_pull_secrets_offline_token` - An OCP Pull Secrets offline token, loaded via
  the `OCP_PULL_SECRETS_OFFLINE_TOKEN` environment variable when using the provided
  example file. See https://github.com/oasis-roles/ocp_pull_secrets#ocp_pull_secrets
  for more information about this token.

Additionally, these variables are used by the `ocp_install` role to control where
files are written on the remote host:

* `ocp_install_path` - Path to which binaries are installed, such as `openshift-installer`
* `ocp_install_config_dir` - Path to which the install config is generated, i.e. the
  `openshift-installer` configuration directory.

Both paths must be writable by the Ansible user.

[aws](group_vars/aws.example)
---

`group_vars` specific to deployment on the Amazon Web Services platform:

* `aws_config_dir` - directory into which AWS credentials will be copied,
  defaults to "~/.aws" on the remote host
* `aws_credential_ini` - credentials file, in ini format, containing access key ID and
  secret access key for use with this cluster, e.g.

```ini
[default]
aws_access_key_id     = abc
aws_secret_access_key = 123
```

* `ocp_install_platform` - Platform section of OCP 4 `install-config.yml` for this platform, e.g.
```yaml
ocp_install_platform:
  aws:
    # name of resource group containing base domain dns zone
    region: exampleregion
```

For reference, consult the OpenShift
[AWS Install Docs](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_aws/installing-on-aws)

[azure](group_vars/azure.example)
-----

`group_vars` specific to deployment on the Microsoft Azure platform:

* `az_config_dir` - directory into which Azure credentials will be copied, defaults to
  "~/.azure/" on the remote host
* `az_os_service_principal_file` - Existing service principal JSON file on control
  machine, containing credentials for Azure OCP deployment, e.g.

```json
{
    "subscriptionId": "...",
    "tenantId": "...",
    "clientId": "...",
    "clientSecret": "..."
}
```

* `ocp_install_platform` - Platform section of OCP 4 `install-config.yml` for this platform, e.g.
```yaml
ocp_install_platform:
  # "azure" platform key is required for all azure deployments
  azure:
    # name of resource group containing base domain dns zone
    baseDomainResourceGroupName: resource-group
    region: exampleregion
```

[This guide](https://docs.microsoft.com/en-us/azure/automation/automation-create-standalone-account)
outlines the creating of Automation Accounts and an associated Service Principal that can be used
to automate deployments on this platform (such as for CI testing). Once the service principal is
created, its App ID can be used as the Service Principal name when following the OCP
[Azure Install Docs](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_azure/installing-on-azure#installation-azure-service-principal_installing-azure-account),
starting at step 4 to get the "password" value for that service principal after the `create-for-rbac`
call.

[gcp](group_vars/gcp.example)
---

# group vars for an IPI deployment in GCP

* `gcp_config_dir` - directory into which Google Cloud Platform credentials will be copied,
  default to "~/.gcp" on the remote host
* `gcp_service_account_file` - Existing GCP service account JSON file on control machine,
  containing credentials for OCP deployment on GCP, e.g.

```json
{
  "type": "service_account",
  "project_id": "your-project-123456",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "...",
  "client_id": "...",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/..."
}
```

* `ocp_install_platform` - Platform section of OCP 4 `install-config.yml` for this platform, e.g.

```yaml
ocp_install_platform:
  # "gcp" platform key is required for all gcp deployments
  gcp:
    # project ideally matches the project identified in the
    # gcp service account json
    projectID: your-project-123456
    region: exampleregion
```

For reference, consult the OpenShift
[GCP Install Docs](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_gcp/installing-on-gcp)

Usage and Examples
==================

After copying the `*.example` `group_vars` files to their non-example locations,
and editing them to provide the expected values for a given platform, creating an
OpenShift 4 installation on your platform of choice should happen with a single
`ansible-playbook` call:

`ansible-playbook -i <inventory> playbooks/site.yml`

The host used in the inventory can be nearly any host with SSH access, or even
the local host (127.0.0.1) provided the paths used are writable by the calling
user.

After deployment, the cluster should be available at the configured endpoint.
As with a normal deployments, the kubernetes credentials are available on the remote
host in `{{ ocp_install_config_dir }}/auth/kubeconfig`.
