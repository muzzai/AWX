# provision_windows_ssh

Role that provisions an AWS EC2 Windows Server 2025 instance capable of OpenSSH administration only. It is optimized for use within AWX and integrates with HashiCorp Vault KV v2 through AppRole authentication.

## Role Variables
| Variable | Default | Description |
| --- | --- | --- |
| `instanceName` | `awx-win-ssh` | Canonical name for the instance; drives Name tag, security-group name, Route53 record, and Vault secret path. |
| `aws_region` | `us-east-2` | AWS region that hosts the EC2 instance. |
| `aws_instance_type` | `m6i.large` | Instance type to create. |
| `aws_subnet_id` | **required** | Subnet where the instance is launched. |
| `aws_security_group_ids` | `[]` | Pre-existing security group IDs to attach. Required when `create_security_group` is false. |
| `create_security_group` | `false` | Toggle to create a dedicated SSH security group (`<instanceName>-ssh`). Requires `aws_vpc_id`. |
| `ssh_ingress_cidr_blocks` | `['0.0.0.0/0']` | CIDR blocks allowed to reach TCP/22 when creating a security group. |
| `assign_public_ip` | `true` | Assigns a public IP during launch. Set to false for private-only subnets. |
| `aws_ami_id` | `` | AMI ID to launch (set to a Windows Server 2025 image). |
| `aws_tags` | `{Project: AWX-Windows-SSH, ...}` | Base tag map merged with `{'Name': instanceName}` at runtime. |
| `route53_enabled` | `false` | Creates an A record when true. Record defaults to `<instanceName>.<route53_zone_name>`. |
| `route53_zone_name` | `example.com.` | Hosted zone name (optional when `route53_zone_id` set). |
| `route53_zone_id` | `""` | Hosted zone ID used when provided. |
| `route53_record` | `""` | Optional override for the Route53 record name. |
| `vault_ca_bundle` | `{{ playbook_dir }}/../artifacts/vault-ca-bundle.pem` | Location where the CA bundle should exist or be written during the play. |
| `vault_kv_mount` | `secret` | KV v2 mount containing the secret path. |
| `vault_secret_path` | `""` | Optional override for the KV v2 path; defaults to `windows/ssh/<instanceName>`. |
| `vault_secret_metadata` | `{}` | Extra metadata stored alongside the key material. |
| `windows_admin_username` | `Administrator` | Account that gains SSH/RDP access. |
| `windows_temporary_password` | `""` | Optional temporary password applied to the Administrator account. |
| `windows_enable_rdp` | `false` | Enables or disables Remote Desktop. When true, firewall rules are opened and the supplied password is used for RDP. |
| `wait_for_ssh_timeout` | `900` | Seconds to wait for the OpenSSH service to accept connections. |

AppRole credentials and Vault connection details are expected via environment variables (`VAULT_ROLE_ID`, `VAULT_SECRET_ID`, `VAULT_ADDR`, optional `VAULT_NAMESPACE`, and either `VAULT_CACERT` or inline `VAULT_CA_CERT_PEM`). Override the variables above only when deviating from those defaults.

## Outputs
The role reports the instance ID, IP addresses, Route53 record (when enabled), and the Vault KV v2 path containing the stored key material. The private key never remains on disk after the play completes.
