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
| `ami_ssm_parameter` | `/aws/service/ami-windows-latest/Windows_Server-2025-English-Full-Base` | SSM parameter used to resolve the AMI ID. |
| `aws_tags` | `{Project: AWX-Windows-SSH, ...}` | Base tag map merged with `{'Name': instanceName}` at runtime. |
| `route53_enabled` | `false` | Creates an A record when true. Record defaults to `<instanceName>.<route53_zone_name>`. |
| `route53_zone_name` | `example.com.` | Hosted zone name (optional when `route53_zone_id` set). |
| `route53_zone_id` | `""` | Hosted zone ID used when provided. |
| `route53_record` | `""` | Optional override for the Route53 record name. |
| `key_output_dir` | `{{ playbook_dir }}/artifacts/ssh_keys` | Temporary location for generated keyfiles. |
| `regenerate_keypair` | `true` | Always generate a fresh key on each run when true. |
| `vault_addr` | `https://172.31.14.234:8200` | Vault endpoint URL. |
| `vault_ca_bundle` | `/etc/vault.d/tls/vault-ca-bundle.pem` | CA bundle path on the controller. |
| `vault_namespace` | `""` | Optional Vault namespace. Omitted when empty. |
| `vault_kv_mount` | `secret` | KV v2 mount containing the secret path. |
| `vault_secret_path` | `""` | Optional override for the KV v2 path; defaults to `windows/ssh/<instanceName>`. |
| `windows_admin_username` | `Administrator` | Account that gains SSH/RDP access. |
| `windows_temporary_password` | `""` | Optional temporary password applied to the Administrator account. |
| `windows_enable_rdp` | `false` | Enables or disables Remote Desktop. When true, firewall rules are opened and the supplied password is used for RDP. |
| `wait_for_ssh_timeout` | `900` | Seconds to wait for the OpenSSH service to accept connections. |

AppRole credentials are passed in at runtime using the environment variables `VAULT_ROLE_ID` and `VAULT_SECRET_ID`.

## Outputs
The role reports the instance ID, IP addresses, SSH username, Route53 record (when enabled), and the Vault KV v2 path containing the stored key material. The private key never remains on disk after the play completes.
