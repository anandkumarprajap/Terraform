# Terraform + AWS Hands-on Lab: S3, DynamoDB, and EC2

This guide organizes your code into a working Terraform lab, explains each command step by step, and covers the AWS IAM permissions required.

You will practice:

- Installing Terraform and AWS CLI on Ubuntu.
- Configuring AWS credentials and verifying IAM access.
- Creating an S3 bucket and DynamoDB table.
- Configuring an S3 remote backend for Terraform state and state locking.
- Creating an EC2 instance, SSH key pair, security group, and Ubuntu AMI.
- Running `terraform init`, `plan`, `apply`, and `destroy`.

Important: Your S3 bucket and DynamoDB table are configured as Terraform's remote state backend. They must exist before Terraform can initialize that backend. We will create them in a separate bootstrap step first.

## 1. Understand the architecture

Ubuntu terminal

Terraform + AWS CLI + IAM credentials

Terraform configuration

terraform.tf · main.tf · ec2.tf

S3 bucket

Stores remote Terraform state

State locking

S3 lock file with `use_lockfile = true`

EC2 instance

Ubuntu · `t3.micro`

Security group

SSH and HTTP inbound rules

Backend note: Your configuration uses `use_lockfile = true`, which enables S3-native state locking in supported Terraform versions. The `dynamodb_table` argument is deprecated in newer AWS provider/backend workflows and is not required for S3 lock-file locking. For this lab, we'll use S3 locking only and create the DynamoDB table separately to practice DynamoDB resources.

## 2. Install Terraform on Ubuntu

Run these commands on your Ubuntu EC2 instance or Ubuntu machine.

### Step 1: Add the HashiCorp repository

```bash
sudo apt-get update
sudo apt-get install -y gnupg software-properties-common wget
```

Download and install the HashiCorp signing key:

```bash
wget -O- https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor \
  -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

Add the repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

### Step 2: Install and verify Terraform

```bash
sudo apt-get update
sudo apt-get install -y terraform
terraform -version
```

Expected output: a Terraform version number. Your installed version should satisfy the version requirement in your configuration.

## 3. Install AWS CLI v2

### Step 1: Download

```bash
sudo apt-get update
sudo apt-get install -y curl unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

This download is for Linux x86-64. For an ARM-based machine, use the appropriate AWS CLI ARM64 installer.

### Step 2: Extract the archive

```bash
unzip awscliv2.zip
```

### Step 3: Install AWS CLI

```bash
sudo ./aws/install
```

### Step 4: Verify

```bash
aws --version
```

Expected output starts with `aws-cli/2`.

## 4. Configure AWS access and IAM permissions

### Step 1: Configure credentials

For a learning machine using IAM access keys:

```bash
aws configure
```

Enter your own values when prompted:

```bash
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_ACCESS_KEY
Default region name: us-east-2
Default output format: json
```

Never put real access keys in your README, Terraform files, GitHub repository, or screenshots.

If your Ubuntu machine is an EC2 instance, prefer an attached IAM role instead of long-lived access keys. In that case, skip `aws configure` if the role is already configured correctly.

### Step 2: Verify your identity

```bash
aws sts get-caller-identity
```

This returns your AWS account ID and the IAM user or role associated with the active credentials.

### Step 3: Verify the region and S3 access

```bash
aws configure get region
aws s3 ls
```

The configured region should be `us-east-2`. The S3 listing will work only if the active identity has the appropriate permissions.

### Step 4: IAM permissions required

For your complete lab, the identity needs permissions appropriate to the resources it will create and manage.

| Service             | Example permissions needed                                                                      |
| ------------------- | ----------------------------------------------------------------------------------------------- |
| S3 backend          | List the designated bucket; read/write/delete state and lock objects                            |
| DynamoDB            | Create, describe, read, update, and delete the practice table, if managing it through Terraform |
| EC2                 | Create, describe, modify, and terminate instances                                               |
| VPC/security groups | Describe VPCs and manage security groups and their ingress/egress rules                         |
| Key pairs           | Import and describe EC2 key pairs                                                               |
| AMI lookup          | Describe images and related EC2 metadata                                                        |
| STS                 | `GetCallerIdentity` for identity verification                                                   |

Use a dedicated IAM role or user with least-privilege permissions scoped to your lab resources. Avoid using the AWS root account or attaching unrestricted administrator access for routine work. If you get `AccessDenied`, check the exact denied action, IAM policy, permission boundaries, and any organization-level SCPs.

## 5. Prepare your SSH key pair

Your `ec2.tf` uses:

```bash
public_key = file("terra-auto-server-key.pub")
```

That public-key file must exist in the Terraform project directory.

Generate a key pair if you don't already have one:

```bash
ssh-keygen -t ed25519 -f terra-auto-server-key
```

Press Enter to accept the file path, and set a passphrase if appropriate.

The generated files are:

- `terra-auto-server-key` — private key; never commit it to Git.
- `terra-auto-server-key.pub` — public key; Terraform imports this into AWS.

Protect the private key:

```bash
chmod 600 terra-auto-server-key
```

If you already have a compatible SSH key pair, use its existing public key instead of generating another one.

## 6. Create the Terraform project

Create a dedicated directory:

```bash
mkdir terraform-aws-lab
cd terraform-aws-lab
```

Use this structure:

```txt
terraform-aws-lab/
├── terraform.tf
├── main.tf
├── ec2.tf
├── outputs.tf
├── terra-auto-server-key.pub
└── .gitignore
```

The S3 backend bucket must exist before you run `terraform init` with the remote backend. We will bootstrap the backend resources first.

## 7. Bootstrap the S3 bucket and DynamoDB table

Create a separate directory for the initial backend resources:

```
mkdir -p ../terraform-backend-bootstrap
cd ../terraform-backend-bootstrap
```

Create a file named `main.tf`:

```hcl
terraform {
  required_version = ">= 1.15.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.59.0"
    }
  }
}

provider "aws" {
  region = "us-east-2"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "anandkumarprajap-bucket"

  lifecycle {
    prevent_destroy = true
  }

  tags = {
    Name    = "terraform-state"
    Purpose = "Terraform remote state"
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_dynamodb_table" "state_table" {
  name         = "anandkumarprajap_table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name    = "terraform-state-practice"
    Purpose = "DynamoDB learning exercise"
  }
}
```

Important: S3 bucket names are globally unique. If the bucket name is already taken by another AWS account, change it to a unique name and use that same name in your backend configuration.

Run these commands from `terraform-backend-bootstrap`:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Type `yes` when prompted to approve the changes.

Verify the resources:

```bash
aws s3 ls
aws dynamodb describe-table \
  --table-name anandkumarprajap_table \
  --region us-east-2
```

The bucket now has versioning enabled, which helps recover previous versions of the Terraform state object.

## 8. Configure the main Terraform project

Return to your project directory:

```bash
cd ../terraform-aws-lab
```

### File 1: `terraform.tf`

Use this backend configuration for S3-native locking:

```hcl
terraform {
  required_version = ">= 1.15.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.59.0"
    }
  }

  backend "s3" {
    bucket       = "anandkumarprajap-bucket"
    key          = "terraform.tfstate"
    region       = "us-east-2"
    use_lockfile = true
  }
}

provider "aws" {
  region = "us-east-2"
}
```

Why is `dynamodb_table` omitted here? Your configuration enables `use_lockfile = true`, which uses an S3 lock file. The DynamoDB table is created separately for practice; it is not needed for this backend's locking. The deprecated `dynamodb_table` backend argument should not be added to a new S3-lockfile-only configuration.

### File 2: `main.tf`

This file creates a DynamoDB table only if you want Terraform in the main project to manage it too. Because the table was already created by the bootstrap configuration, do not duplicate the resource under the same name.

For now, leave `main.tf` in the main project empty or use it for additional resources later. The bootstrap configuration owns the bucket and DynamoDB table.

### File 3: `ec2.tf`

Use your EC2 configuration with a few formatting improvements:

```hcl
resource "aws_key_pair" "my_ec2_key" {
  key_name   = "terra-auto-server-key"
  public_key = file("${path.module}/terra-auto-server-key.pub")
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_default_vpc" "default" {}

resource "aws_security_group" "my_ec2_sg" {
  name        = "terra-auto-sg"
  description = "Allow SSH and HTTP inbound traffic"
  vpc_id      = aws_default_vpc.default.id

  tags = {
    Name = "terra-auto-sg"
  }
}

resource "aws_vpc_security_group_ingress_rule" "allow_ssh" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 22
  ip_protocol       = "tcp"
  to_port           = 22
}

resource "aws_vpc_security_group_ingress_rule" "allow_http" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 80
  ip_protocol       = "tcp"
  to_port           = 80
}

resource "aws_vpc_security_group_egress_rule" "allow_all_traffic_ipv4" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "-1"
}

resource "aws_instance" "my_ec2_instance" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t3.micro"
  key_name               = aws_key_pair.my_ec2_key.key_name
  vpc_security_group_ids = [aws_security_group.my_ec2_sg.id]

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }

  tags = {
    Name = "terra-auto-server"
  }
}
```

Security note: The SSH rule above allows access from any IPv4 address (`0.0.0.0/0`). For a safer lab, replace it with your own public IP in CIDR format, for example `YOUR_PUBLIC_IP/32`. HTTP is also open to the internet. An EC2 instance may incur charges, so check current AWS pricing and your account's free-tier eligibility.

### File 4: `outputs.tf`

```hcl
output "ec2_instance_id" {
  description = "ID of the created EC2 instance"
  value       = aws_instance.my_ec2_instance.id
}

output "ec2_public_ip" {
  description = "Public IPv4 address, if assigned"
  value       = aws_instance.my_ec2_instance.public_ip
}

output "ec2_public_dns" {
  description = "Public DNS name, if assigned"
  value       = aws_instance.my_ec2_instance.public_dns
}
```

### File 5: `.gitignore`

```.gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
*.pem
terra-auto-server-key
awscliv2.zip
```

Keep `.terraform.lock.hcl` under version control. Never commit private keys, AWS credentials, or Terraform state files.

## 9. Initialize and deploy the EC2 infrastructure

Run the commands from `terraform-aws-lab`.

### Step 1: Initialize Terraform

```bash
terraform init
```

Terraform downloads the required provider and initializes the S3 backend. If the backend is not accessible, check that the bucket exists, the region is correct, and your IAM identity has the required S3 permissions.

### Step 2: Format the configuration

```bash
terraform fmt -recursive
```

This standardizes indentation and formatting.

### Step 3: Validate the configuration

```bash
terraform validate
```

This checks the configuration's syntax and structure. It does not guarantee that AWS permissions or resources are available.

### Step 4: Review the execution plan

```bash
terraform plan
```

Review the resources Terraform intends to create, change, or destroy before proceeding.

### Step 5: Apply the configuration

```bash
terraform apply
```

Type `yes` only after reviewing the plan.

### Step 6: Verify the deployment

```bash
terraform output
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=terra-auto-server" \
  --region us-east-2
```

The instance should appear in the AWS console under EC2 → Instances, in the `us-east-2` region.

## 10. Test SSH access

Retrieve the public IP:

```bash
terraform output -raw ec2_public_ip
```

Connect using your private key:

```bash
ssh -i terra-auto-server-key ubuntu@YOUR_EC2_PUBLIC_IP
```

Replace `YOUR_EC2_PUBLIC_IP` with the actual address. If the instance has no public IP or cannot be reached, verify its subnet, routing, internet gateway, security group, and key pair. Your security group permits inbound HTTP, but the instance will not serve a website unless a web server is installed and running.

## 11. Common errors and fixes

| Error                                         | What to check                                                          |
| --------------------------------------------- | ---------------------------------------------------------------------- |
| `terraform: command not found`                | Installation and executable path                                       |
| `Unable to locate credentials`                | AWS CLI configuration or attached EC2 IAM role                         |
| `AccessDenied`                                | IAM policy, role, permission boundary, or organization SCP             |
| S3 bucket does not exist                      | Bootstrap the backend before running `terraform init`                  |
| Bucket name already exists                    | Choose a globally unique bucket name                                   |
| Public key file not found                     | Confirm `terra-auto-server-key.pub` exists in the project directory    |
| SSH connection timed out                      | Public IP, routing, security group, and instance status                |
| `terraform init` asks about backend migration | Review the migration prompt; back up state and migrate only when ready |

## 12. Clean up resources

When you finish the EC2 lab, remove the resources managed by the main project:

```bash
terraform destroy
```

Review the plan and type `yes` to confirm. This deletes the EC2 instance, key-pair registration, and security-group resources managed by that configuration.

Do not destroy the bootstrap project casually. The S3 bucket contains remote Terraform state. If you need to remove the backend later, first migrate or back up the state and confirm no other configuration depends on it. The bootstrap bucket's `prevent_destroy` lifecycle rule also blocks accidental deletion through Terraform until deliberately changed.

## Save as `README.md`

Use the following final checklist when documenting your hands-on work in GitHub:

- Terraform installed and verified.
- AWS CLI installed and verified.
- AWS identity confirmed with `aws sts get-caller-identity`.
- IAM permissions checked.
- Backend S3 bucket and versioning configured.
- DynamoDB practice table created.
- SSH public key available.
- Terraform initialized successfully.
- Configuration formatted and validated.
- Plan reviewed and EC2 deployed.
- Outputs and SSH access tested.
- Lab resources destroyed when no longer needed.

This workflow separates backend bootstrap from the main EC2 deployment so that Terraform can initialize its remote state before managing the application infrastructure.
   
