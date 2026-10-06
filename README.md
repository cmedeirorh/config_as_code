# Ansible Automation Platform - Configuration as Code

This repository contains Ansible Configuration as Code (CaC) for managing Red Hat Ansible Automation Platform (AAP) resources, including Controller, Event-Driven Ansible (EDA), and Gateway settings.

## ⚠️ Security Notice

This repository uses Jinja2 template variables for all sensitive values. **Never commit actual credentials or secrets to this repository.**

All sensitive values must be provided separately via:
- An Ansible Vault encrypted variables file
- A `vars.yml` file (excluded from git via `.gitignore`)
- An external secrets manager

## Repository Structure

    .
    ├── collections/
    │   └── requirements.yml              # Required Ansible collections
    ├── config/                           # Production AAP configuration
    │   ├── controller_credential.yml
    │   ├── controller_credential_types.yml
    │   ├── controller_ee.yml
    │   ├── controller_inventories.yml
    │   ├── controller_projects.yml
    │   ├── controller_settings.yml
    │   ├── controller_templates.yml
    │   ├── eda_credentials.yml
    │   ├── eda_decision_environments.yml
    │   ├── eda_event_streams.yml
    │   ├── eda_projects.yml
    │   ├── eda_rulebook_activations.yml
    │   ├── gateway_organizations.yml
    │   └── gateway_settings.yml
    ├── config_workshop/                  # Workshop AAP configuration
    │   ├── controller_credential.yml
    │   ├── controller_credential_types.yml
    │   ├── controller_ee.yml
    │   ├── controller_inventories.yml
    │   ├── controller_projects.yml
    │   ├── controller_settings.yml
    │   ├── controller_templates.yml
    │   ├── eda_credentials.yml
    │   ├── eda_decision_environments.yml
    │   ├── eda_event_streams.yml
    │   ├── eda_projects.yml
    │   └── eda_rulebook_activations.yml
    ├── config_aiops_workshop/            # AIOps Workshop AAP configuration
    │   └── controller_projects.yml
    └── playbooks/
        ├── setup_aap.yml                 # Full production AAP setup
        ├── setup_workshop.yml            # Workshop AAP setup
        ├── setup_aiops_workshop.yml      # AIOps workshop AAP setup
        └── activate_subs.yml             # AAP license activation only

## Prerequisites

- Ansible installed on your local machine
- Access to a running Red Hat Ansible Automation Platform instance
- Required Ansible collections (see [Collections](#collections))
- A valid AAP manifest file
- Required variables defined (see [Required Variables](#required-variables))

## Collections

Install the required collections using:

`ansible-galaxy collection install -r collections/requirements.yml`

The following collections are required:

| Collection | Version | Description |
|---|---|---|
| `ansible.platform` | latest | Ansible Platform modules |
| `ansible.controller` | latest | Ansible Controller modules |
| `infra.aap_configuration` | 4.7.0 | AAP Configuration as Code roles |

## Required Variables

Create a `vars.yml` file (it is excluded from git) with the following variables:

    # AAP Connection
    aap_hostname: "https://your-aap-hostname"
    aap_username: "admin"
    aap_password: "your-aap-password"

    # AAP Manifest
    manifest_url: "https://url-to-your-manifest"
    manifest_local: "/tmp/manifest.zip"

    # Organization
    my_organization: "Your Organization"

    # Red Hat Subscription
    redhat_client_id: "your-client-id"
    redhat_client_secret: "your-client-secret"

    # Red Hat Registry
    rh_registry_username: "your-registry-username"
    rh_registry_token: "your-registry-token"

    # Red Hat Automation Hub
    red_hat_auto_hub_token: "your-automationhub-token"

    # AWS Credentials
    aws_access_key: "your-aws-access-key"
    aws_secret_key: "your-aws-secret-key"

    # Machine Credentials
    ssh_username: "your-ssh-username"
    ssh_priv_key: |
      -----BEGIN OPENSSH PRIVATE KEY-----
      your-private-key-here
      -----END OPENSSH PRIVATE KEY-----

    # ServiceNow
    snow_instance: "https://your-instance.service-now.com"
    snow_username: "your-snow-username"
    snow_password: "your-snow-password"

    # EDA
    eda_event_stream_token: "your-eda-event-stream-token"

    # Workshop specific
    uuid: "your-workshop-uuid"

## Usage

### Full Production AAP Setup

Configures all production AAP resources including Controller, EDA, and Gateway:

`ansible-playbook playbooks/setup_aap.yml -e @vars.yml`

### Workshop AAP Setup

Configures AAP resources for a workshop environment:

`ansible-playbook playbooks/setup_workshop.yml -e @vars.yml`

### AIOps Workshop AAP Setup

Configures AAP resources for an AIOps workshop environment:

`ansible-playbook playbooks/setup_aiops_workshop.yml -e @vars.yml`

### License Activation Only

Activates the AAP license without applying any other configuration:

`ansible-playbook playbooks/activate_subs.yml -e @vars.yml`

## Configuration Details

### Controller Configuration

- **Credentials** - AWS, Machine, ServiceNow, Container Registry, and Automation Hub credentials
- **Credential Types** - Custom credential types for ServiceNow ITSM and Dynatrace API
- **Execution Environments** - Container images used for job execution (`quay.io/redhat_emp1/cmedeiro-ee-general:latest`)
- **Inventories** - Dynamic AWS EC2 inventory with tag-based filtering
- **Projects** - Source control projects linked to GitHub
- **Templates** - Job templates for RHEL management, patching, and ServiceNow integration
- **Settings** - Controller application settings including Red Hat subscription configuration

### EDA Configuration

- **Credentials** - Controller and event stream token credentials
- **Decision Environments** - Container images for EDA (`registry.redhat.io/ansible-automation-platform-26/de-supported-rhel9:latest`)
- **Event Streams** - Incoming event stream configurations with token authentication
- **Projects** - EDA source control projects linked to GitHub
- **Rulebook Activations** - Active rulebook configurations for ServiceNow event monitoring

### Gateway Configuration

- **Organizations** - Gateway organization definitions with Automation Hub credentials
- **Settings** - Gateway application settings including session management and subscription configuration

### Job Templates

| Template | Description |
|---|---|
| RHEL Sync SSH Keys | Synchronizes SSH keys across RHEL instances |
| RHEL Deploy APP With Podman | Deploys containerized applications using Podman |
| RHEL SSH Hardening | Applies SSH hardening using DevSec profile |
| RHEL ServiceNow - Check for Updates | Checks for RHEL updates and creates ServiceNow records |
| RHEL ServiceNow - Apply Updates | Applies RHEL updates and updates ServiceNow records |

## Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes, ensuring no secrets or credentials are hardcoded
4. Submit a pull request

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.
