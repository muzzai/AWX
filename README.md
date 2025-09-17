# AWX Windows SSH Provisioner

This repository contains an Ansible project built for AWX that provisions an AWS EC2 Windows Server 2025 (English, Full, Base) instance configured for OpenSSH-only administration. It generates an ED25519 keypair, stores the private key in HashiCorp Vault KV v2 via AppRole, and optionally manages Route53 DNS.

## Layout
- `ansible.cfg` — controller defaults tuned for AWX execution environments.
- `collections/requirements.yml` — required Ansible collections (`amazon.aws`, `community.aws`, `community.hashi_vault`, `community.crypto`).
- `inventories/localhost.yml` — localhost inventory stub for provisioning from the controller.
- `group_vars/all.yml` — opinionated defaults; override per environment or via AWX survey.
- `playbooks/provision-windows-ssh.yml` — entry-point playbook.
- `roles/provision_windows_ssh/` — encapsulates provisioning logic and Windows bootstrap user data.
- `docs/` — extended usage notes.

## Quick Start (local testing)
1. Install Python dependencies: `pip install boto3 botocore hvac`.
2. Install collections: `ansible-galaxy collection install -r collections/requirements.yml`.
3. Export Vault AppRole credentials:
   ```bash
   export VAULT_ROLE_ID=... VAULT_SECRET_ID=...
   ```
4. Run the playbook with the required AWS inputs:
   ```bash
   ansible-playbook playbooks/provision-windows-ssh.yml \
     -e aws_subnet_id=subnet-1234567890abcdef \
     -e aws_security_group_ids='["sg-1234567890abcdef"]' \
     -e vault_secret_path=windows/ssh/prod-admin
   ```

When the job finishes, retrieve the private key from Vault (`secret/data/windows/ssh/prod-admin` by default) and connect using `ssh -i` against the reported host/IP.
