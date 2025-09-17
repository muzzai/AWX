# Provisioning Windows Server 2025 over SSH

This Ansible project provisions a hardened Windows Server 2025 (English, Full, Base) EC2 instance that is manageable exclusively via OpenSSH. It is intended to run under AWX using an AWS credential (Instance Profile or IAM keys) and a HashiCorp Vault AppRole for secret storage.

## Features
- Resolves the latest Windows Server 2025 AMI from AWS SSM Parameter Store.
- Boots with user data that enables the OpenSSH Server feature, hardens `administrators_authorized_keys`, and disables password logins.
- Generates an ephemeral ED25519 keypair, loads the public key via user data, and writes the private key and metadata to Vault KV v2 using a custom CA bundle and AppRole credentials.
- Optionally creates a managed security group and Route53 A record for the instance.
- Waits for TCP/22 to respond so follow-up jobs can immediately connect with SSH.

## Prerequisites
- Ansible 2.16 or newer with `boto3`, `botocore`, and `hvac` installed.
- Collections from `collections/requirements.yml` installed (`ansible-galaxy collection install -r collections/requirements.yml`).
- AWX project set to inject `VAULT_ROLE_ID` and `VAULT_SECRET_ID` environment variables and trust the Vault CA bundle located at `/etc/vault.d/tls/vault-ca-bundle.pem` on the controller.
- IAM permissions to create EC2 instances, security groups (if `create_security_group`), Route53 records (if enabled), and read the SSM public parameter.

## Usage in AWX
1. Sync this repository into an AWX project with execution environment that includes the dependencies above.
2. Create or assign AWS credentials (instance profile or IAM key) to the job template.
3. Create a HashiCorp Vault credential using AppRole auth that exposes `VAULT_ROLE_ID` and `VAULT_SECRET_ID`.
4. Supply runtime variables (e.g., `aws_subnet_id`, `aws_security_group_ids` or `create_security_group: true`, `vault_secret_path`) through a survey or extra vars.
5. Launch the job with playbook `playbooks/provision-windows-ssh.yml`. On completion the private key is stored at `{{ vault_kv_mount }}/data/{{ vault_secret_path }}` and the instance details are echoed in the job output.
