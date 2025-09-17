# Provisioning Windows Server 2025 over SSH

This Ansible project provisions a hardened Windows Server 2025 (English, Full, Base) EC2 instance that is manageable exclusively via OpenSSH. It is intended to run under AWX using an AWS credential (Instance Profile or IAM keys) and a HashiCorp Vault AppRole for secret storage.

## Features
- Resolves the latest Windows Server 2025 AMI from AWS SSM Parameter Store.
- Boots with user data that enables the OpenSSH Server feature, hardens `administrators_authorized_keys`, optionally applies a temporary Administrator password, and enforces/blocks RDP based on policy.
- Generates an ephemeral ED25519 keypair, loads the public key via user data, and writes the private key and metadata to Vault KV v2 using AppRole credentials supplied at runtime.
- Names security groups, DNS records, and Vault paths automatically from a single `instanceName` variable, with optional Route53 zone ID support.
- Waits for TCP/22 to respond so follow-up jobs can immediately connect with SSH.

## Prerequisites
- Ansible 2.16 or newer with `boto3`, `botocore`, and `hvac` installed.
- Collections from `collections/requirements.yml` installed (`ansible-galaxy collection install -r collections/requirements.yml`).
- AWX project set to inject `VAULT_ROLE_ID`, `VAULT_SECRET_ID`, and `VAULT_ADDR` environment variables (optionally `VAULT_NAMESPACE`, `VAULT_CACERT`, or inline `VAULT_CA_CERT_PEM`).
- The CA bundle trusted by the Vault server is available to the execution environment; by default the play writes any injected PEM to `{{ playbook_dir }}/../artifacts/vault-ca-bundle.pem`.
- IAM permissions to create EC2 instances, security groups (if `create_security_group`), Route53 records (if enabled), and read the SSM public parameter.

## Usage in AWX
1. Sync this repository into an AWX project with an execution environment that includes the dependencies above.
2. Create or assign AWS credentials (instance profile or IAM key) to the job template.
3. Create a HashiCorp Vault AppRole credential that exports the environment variables mentioned in Prerequisites (or apply them via the job template). Paste the Vault CA PEM into the credential if you want the play to materialize it automatically.
4. Supply runtime variables—at minimum `instanceName`, `aws_subnet_id`, and either `aws_security_group_ids` or `create_security_group: true` (plus `aws_vpc_id`). Optionally set `windows_temporary_password`, `windows_enable_rdp`, and Route53 zone details.
5. Launch the job with playbook `playbooks/provision-windows-ssh.yml`. On completion the private key is stored at `{{ vault_kv_mount }}/data/windows/ssh/<instanceName>` (or the override you supply) and the instance details (including DNS) are echoed in the job output.
