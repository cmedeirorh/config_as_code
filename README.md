# Ansible Automation Platform - Configuration as Code

Declarative Configuration as Code (CaC) for Red Hat Ansible Automation Platform, managing Controller, Event-Driven Ansible (EDA), and Gateway resources via the [`infra.aap_configuration`](https://github.com/redhat-cop/aap_configuration) collection.

## Security notice

All sensitive values are Jinja2 variables. **Never commit credentials or secrets.** The `.gitignore` excludes `vars.yml`, `vars/*.yml`, vault files, and `.env`.

Supply secrets at runtime via:
- A `vars.yml` file (not committed)
- An Ansible Vault encrypted file
- An external secrets manager

## Repository structure

```
.
├── collections/
│   └── requirements.yml              # Required Ansible collections
├── config/                           # Production AAP configuration
│   ├── controller_credential.yml     # Credentials (AWS, Machine, ServiceNow, Hub, Registry)
│   ├── controller_credential_types.yml  # Custom types: ServiceNow ITSM, Dynatrace API Token
│   ├── controller_ee.yml             # Execution environment
│   ├── controller_inventories.yml    # Dynamic AWS EC2 inventory
│   ├── controller_projects.yml       # Source control projects
│   ├── controller_settings.yml       # Controller application settings
│   ├── controller_templates.yml      # Job templates
│   ├── eda_credentials.yml           # EDA credentials
│   ├── eda_decision_environments.yml # EDA decision environments
│   ├── eda_event_streams.yml         # EDA event streams
│   ├── eda_projects.yml              # EDA source control projects
│   ├── eda_rulebook_activations.yml  # EDA rulebook activations
│   ├── gateway_organizations.yml     # Gateway organizations
│   └── gateway_settings.yml          # Gateway application settings
├── config_workshop/                  # Workshop AAP configuration
│   └── (same structure as config/, workshop-specific values)
├── config_aiops_workshop/            # AIOps workshop configuration
│   └── controller_projects.yml
└── playbooks/
    ├── setup_aap.yml                 # Full production setup (includes license activation)
    ├── setup_workshop.yml            # Workshop setup (requires pre-licensed controller)
    ├── setup_aiops_workshop.yml      # AIOps workshop setup (requires pre-licensed controller)
    └── activate_subs.yml             # License activation only
```

## Prerequisites

- Ansible installed locally
- A running AAP instance
- Required collections installed (see [Collections](#collections))
- A valid AAP manifest file accessible at `manifest_url`
- Variables file with all required values (see [Required Variables](#required-variables))

## Collections

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

| Collection | Version | Purpose |
|---|---|---|
| `ansible.platform` | latest | Ansible Platform modules |
| `ansible.controller` | latest | Controller modules |
| `infra.aap_configuration` | 4.7.0 | CaC dispatch role |

## Required variables

Create a `vars.yml` file in the project root (excluded from git):

```yaml
# AAP connection
aap_hostname: "https://your-aap-hostname"
aap_username: "admin"
aap_password: "your-aap-password"

# AAP manifest (required for setup_aap.yml and activate_subs.yml)
manifest_url: "https://url-to-your-manifest-file"
manifest_local: "/tmp/manifest.zip"

# Organization
my_organization: "Your Organization"

# Red Hat subscription
redhat_client_id: "your-client-id"
redhat_client_secret: "your-client-secret"

# Red Hat registry
rh_registry_username: "your-registry-username"
rh_registry_token: "your-registry-token"

# Red Hat Automation Hub
red_hat_auto_hub_token: "your-automationhub-token"

# AWS credentials
aws_access_key: "your-aws-access-key"
aws_secret_key: "your-aws-secret-key"

# Machine (SSH) credential
ssh_username: "ec2-user"
ssh_priv_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  ...
  -----END OPENSSH PRIVATE KEY-----

# ServiceNow
snow_instance: "https://your-instance.service-now.com"
snow_username: "your-snow-username"
snow_password: "your-snow-password"

# EDA event stream token
eda_event_stream_token: "your-eda-event-stream-token"

# Workshop UUID (used to filter EC2 hosts by owner tag)
uuid: "your-workshop-uuid"
```

## Usage

### Full production setup

Downloads the manifest, activates the AAP license, and applies all configuration from `config/`:

```bash
ansible-playbook playbooks/setup_aap.yml -e @vars.yml
```

### Workshop setup

Applies workshop configuration from `config_workshop/`. Requires the controller to already be licensed — run `activate_subs.yml` first on a fresh instance:

```bash
# 1. License the controller (fresh instances only)
ansible-playbook playbooks/activate_subs.yml -e @vars.yml

# 2. Apply workshop configuration
ansible-playbook playbooks/setup_workshop.yml -e @vars.yml
```

### AIOps workshop setup

Applies AIOps workshop configuration from `config_aiops_workshop/`. Same licensing prerequisite applies:

```bash
ansible-playbook playbooks/setup_aiops_workshop.yml -e @vars.yml
```

### License activation only

```bash
ansible-playbook playbooks/activate_subs.yml -e @vars.yml
```

## Configuration reference

### Job templates

| Template | Playbook | Purpose |
|---|---|---|
| RHEL Sync SSH Keys | `linux/0-rhel_sync_ssh_keys.yml` | Distribute SSH keys to RHEL instances |
| RHEL Deploy APP With Podman | `linux/6-rhel_podman_app_deploy.yml` | Deploy containerized app via Podman |
| RHEL SSH Hardening | `linux/3-rhel_devsec_ssh_hardening.yml` | Apply DevSec SSH hardening profile |
| RHEL ServiceNow - Check for Updates | `linux/1.1-rhel_check_updates_servicenow.yml` | Check for RHEL updates, open ServiceNow record |
| RHEL ServiceNow - Apply Updates | `linux/1.2-rhel_apply_update_servicenow.yml` | Apply updates, resolve ServiceNow record |

### Inventory

A single dynamic inventory (`Workshop Inventory`) is backed by an AWS EC2 source that:
- Filters hosts by the `node.*` name pattern and an `owner` tag matching `{{ uuid }}`
- Sets `ansible_host` from the public IP and `ansible_user` to `ec2-user`
- Groups hosts by OS platform, blueprint tag, and owner tag

### EDA

The `Monitoring Events - ServiceNow` rulebook activation listens on the `Default Event Stream` (token-authenticated) and routes events to Controller job templates.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Verify no credentials or secrets are hardcoded before committing
4. Open a pull request
