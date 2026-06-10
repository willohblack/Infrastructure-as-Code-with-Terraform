# Infrastructure as Code with Terraform

**Cloud:** AWS · **Tools:** Terraform, S3, DynamoDB · **Type:** IaC / DevOps

---

## Overview

Modular Terraform project that provisions a full AWS network and compute baseline: VPC, public/private subnets, security groups, and EC2 instances. Remote state stored in S3 with DynamoDB locking for team safety. Variable-driven design supports multiple environments (dev, staging, prod) from a single codebase.

---

## Repository Structure

```
terraform/
├── main.tf              # Root module, calls child modules
├── variables.tf         # Input variable declarations
├── outputs.tf           # Output values (VPC ID, instance IP, etc.)
├── terraform.tfvars     # Environment-specific values (gitignored for prod)
├── backend.tf           # Remote state config (S3 + DynamoDB)
└── modules/
    ├── networking/      # VPC, subnets, IGW, route tables
    ├── security/        # Security groups
    └── compute/         # EC2 launch + key pair
```

---

## Key Components

### Networking Module
- Created VPC with configurable CIDR block
- Public and private subnets across 2 AZs
- Internet Gateway + route table association for public subnets
- NAT Gateway for private subnet outbound access

### Security Module
- Security group for web tier: allow 80/443 inbound, all outbound
- Security group for SSH bastion: restrict to admin CIDR

### Compute Module
- EC2 instance with AMI looked up via `data "aws_ami"` filter (latest Amazon Linux 2)
- User data script to bootstrap Nginx on launch
- Output: public IP and instance ID

### Remote State
```hcl
terraform {
  backend "s3" {
    bucket         = "tf-state-dawood"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-lock"
    encrypt        = true
  }
}
```

### Multi-Environment Design
- All resource names and sizing driven by `var.environment` and `var.instance_type`
- Single `terraform apply -var-file=dev.tfvars` vs `prod.tfvars` to switch environments
- No code duplication across environments

---

## Workflow

```bash
terraform init        # Download providers, configure backend
terraform validate    # Check syntax
terraform plan        # Preview changes
terraform apply       # Provision resources
terraform destroy     # Tear down (dev/test only)
```

---

## Outcome

- Full VPC + EC2 stack provisioned in under 2 minutes from `terraform apply`
- Remote state prevents concurrent apply conflicts in team environments
- Modular structure allows reuse across projects with minimal changes
- Aligned to Terraform best practices: no hardcoded values, outputs documented

---

## Tags

`Terraform` `IaC` `AWS` `VPC` `EC2` `S3 State` `DynamoDB Locking` `DevOps` `Multi-Environment`
