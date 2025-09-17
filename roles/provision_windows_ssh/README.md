# provision_windows_ssh

Role that provisions an AWS EC2 Windows Server 2025 instance capable of OpenSSH administration only. It is optimized for use within AWX and integrates with HashiCorp Vault KV v2 through AppRole authentication.

## Role Variables
| Variable | Default | Description |
| --- | --- | --- |
| `aws_region` | `us-east-2` | AWS region that hosts the EC2 instance. |
| `aws_instance_name` | `awx-win-ssh` | Name tag applied to the instance. |
| `aws_instance_type` | `m6i.large` | Instance type to create. |
| `aws_subnet_id` | **required** | Subnet where the instance is launched. |
| `aws_security_group_ids` | `[]` | Pre-existing security group IDs to attach. Required when `create_security_group` is false. |
| `create_security_group` | `false` | Toggle to create a dedicated SSH security group. Requires `aws_vpc_id`. |
| `ssh_ingress_cidr_blocks` | `['0.0.0.0/0']` | CIDR blocks allowed to reach TCP/22 when creating a security group. |
| `assign_public_ip` | `true` | Assigns a public IP during launch. Set to false for private-only subnets. |
| `ami_ssm_parameter` | `/aws/service/ami-windows-latest/Windows_Server-2025-English-Full-Base` | SSM parameter used to resolve the AMI ID. |
| `vault_addr` | `https://172.31.14.234:8200` | Vault endpoint URL. |
| `vault_ca_bundle` | `/etc/vault.d/tls/vault-ca-bundle.pem` | CA bundle path on the controller. |
| `vault_namespace` | `""` | Optional Vault namespace. Omitted when empty. |
| `vault_kv_mount` | `secret` | KV v2 mount containing the secret path. |
| `vault_secret_path` | `windows/ssh/{{ aws_instance_name }}` | KV v2 path where the private key is written. |
| `route53_enabled` | `false` | Creates an A record when true. |
| `route53_zone` | `example.com.` | Hosted zone name. |
| `route53_record` | `{{ aws_instance_name }}.{{ route53_zone }}` | Record name to manage. |
| `wait_for_ssh_timeout` | `900` | Seconds to wait for the OpenSSH service to accept connections. |
| `regenerate_keypair` | `true` (via group vars) | Always generate a fresh key on each run when true. |

AppRole credentials are passed in at runtime using the environment variables `VAULT_ROLE_ID` and `VAULT_SECRET_ID`.

## Outputs
The role reports the instance ID, IP addresses, SSH username, and the Vault KV v2 path containing the stored key material. The private key never remains on disk after the play completes.
