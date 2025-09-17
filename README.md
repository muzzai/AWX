# AWX Windows SSH Provisioner

This repository contains an Ansible project built for AWX that provisions an AWS EC2 Windows Server 2025 (English, Full, Base) instance configured for OpenSSH-only administration. It derives resource names from a single `instanceName`, generates an ED25519 keypair, stores the private key in HashiCorp Vault KV v2 via AppRole, and optionally manages Route53 DNS.

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
3. Export Vault connection details (match what AWX injects):
   ```bash
   export VAULT_ROLE_ID=...
   export VAULT_SECRET_ID=...
   export VAULT_ADDR=https://172.31.14.234:8200
   export VAULT_CACERT=$(pwd)/artifacts/vault-ca-bundle.pem   # or set VAULT_CA_CERT_PEM with the PEM contents
   ```
4. Run the playbook with the required AWS inputs:
   ```bash
   ansible-playbook playbooks/provision-windows-ssh.yml \
     -e instanceName=prod-awx-win \
     -e aws_ami_id=ami-0123456789abcdef0 \
     -e aws_subnet_id=subnet-1234567890abcdef \
     -e aws_vpc_id=vpc-1234567890abcdef \
     -e create_security_group=true \
     -e ssh_ingress_cidr_blocks='["203.0.113.0/24"]'
   ```

When the job finishes, retrieve the private key from Vault (`secret/data/windows/ssh/prod-awx-win` by default) and connect using `ssh -i` against the reported host/IP. Enable Route53 by setting `route53_enabled: true` and providing either `route53_zone_name` or `route53_zone_id`.
