# Setup EC2 Collection and Authentication

## Install boto3

```bash
pip install boto3
```

## Install AWS Collection

```bash
ansible-galaxy collection install amazon.aws
```

## Setup Vault

1. Create a password for vault
```bash
openssl rand -base64 2048 > vault.pass
```

2. Add your AWS credentials using the below vault command
```bash
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```
3. Populate Vault Variables

Inside your default terminal text editor, structure your AWS keys using double quotes (`""`) to escape special YAML string formatting characters like `/` and `+`:

```yaml
---
ec2_access_key: "YOUR_AWS_ACCESS_KEY_ID"
ec2_secret_key: "YOUR_AWS_SECRET_ACCESS_KEY"
```

## Configuration Code

### Main Playbook (`ec2_create.yaml`)

This root entrypoint maps execution to your local machine and invokes the nested role parameters:

```yaml
---
- name: Deploy EC2 Instance via Role
  hosts: localhost
  connection: local
  roles:
    - ec2
```

### Role Tasks Execution (`ec2/tasks/main.yml`)

Handles direct communication parameters with AWS infrastructure APIs:

```yaml
---
- name: start an instance with a public IP address
  amazon.aws.ec2_instance:
    name: "ansible-instance"
    instance_type: t3.micro
    security_group: default
    region: us-east-1
    aws_access_key: "{{ ec2_access_key }}"
    aws_secret_key: "{{ ec2_secret_key }}"
    network_interfaces:
      - assign_public_ip: true
    image_id: ami-081b0a6eac00b4f53
```

## Execution

Run the deployment automation block by pointing directly to your local workspace files:

```bash
ansible-playbook -i inventory.ini ec2_create.yaml --vault-password-file ~/vault.pass
```
