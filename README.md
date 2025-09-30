# Ansible Playbook to deploy Chainflip Node

This Ansible playbook automates the deployment and configuration of Chainflip Node. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator keys and secrets. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private keys and environment secrets for Mina from HashiCorp Vault. The keys and secrets follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`testnet/chainflip/encapsulate/node/node_key_file`
`testnet/chainflip/encapsulate/engine/node_key_file`
`testnet/chainflip/encapsulate/engine/signing_key_file`
`testnet/chainflip/encapsulate/engine/ethereum_key_file`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/chainflip-ansible.git
cd chainflip-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for `testnet.yml`

```yaml
---
# maintains testnet inventory grouped by project names
all:
  vars:
    env: testnet
  children:
    chainflip:
      hosts:
        chainflip_node_host:
          ansible_host: validator.chainflip.testnet.encapsulate.xyz
          type: node
        chainflip_engine_host:
          ansible_host: validator.chainflip.testnet.encapsulate.xyz
          type: engine
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port, source URL configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.

**Note**: Sensitive variables such as private keys and passwords are not stored in `vault.yml`. They are dynamically retrieved from HashiCorp Vault during the playbook run.

### Usage

1. First, install the dependencies:

```bash
ansible-galaxy role install -r roles/requirements.yml
ansible-galaxy collection install -r collections/requirements.yml
```

2. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

3. Run the playbook:

- To deploy a chainflip node:

```bash
ansible-playbook setup_node.yml -i inventory/testnet.yml --limit chainflip
```

- To deploy a chainflip engine:

```bash
ansible-playbook setup_engine.yml -i inventory/testnet.yml --limit chainflip
```

**Note**:

If you are migrating your Chainflip validator node to a new server, use this command to rotate the keys and restart the engine and node services.

```bash
chainflip-cli --config-root <config_file_path> rotate
```

**Example Output:**

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.chainflip.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.chainflip.testnet.encapsulate.xyz",
        "Groups: ['chainflip']",
        "Project: chainflip'",
        "Environment: testnet",
        "Type: validator",
        "Version: 1.7.5",
        "Service Name: chainflip'",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.chainflip.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.chainflip.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.chainflip.testnet.encapsulate.xyz",
        "Project: chainflip'",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.chainflip.testnet.encapsulate.xyz]
```
