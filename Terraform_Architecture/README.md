# Terraform Hands-On Practice with AWS

## Table of Contents

1. Introduction
2. Architecture and Workflow
3. Install Terraform on Ubuntu EC2
4. Install and Configure AWS CLI
5. Create an IAM User and Configure Permissions
6. Create a Terraform Project
7. Understand HCL Syntax
8. Create an S3 Bucket
9. Initialize, Validate, Plan, and Apply
10. Understand Terraform State
11. Generate an SSH Key Pair
12. Create an EC2 Instance Using Terraform
13. Configure VPC and Security Groups
14. Create Multiple EC2 Instances Using `count`
15. Create a DynamoDB Table
16. Configure an S3 Remote Backend
17. Understand Backend Migration
18. Terraform State Commands
19. Destroy AWS Resources Safely
20. Common Errors and Solutions
21. Best Practices
22. Complete Command Reference

---

## 1. Introduction

**Terraform** is an Infrastructure as Code (IaC) tool developed by HashiCorp. It allows us to create, modify, and manage cloud infrastructure using configuration files.

### Why do we use Terraform?

- Automate AWS resource creation.
- Manage infrastructure using HCL.
- Create EC2 instances, S3 buckets, VPCs, and security groups.
- Track infrastructure using Terraform state.
- Reuse configurations and maintain version control.
- Manage infrastructure changes through a repeatable workflow.

### Technologies used

| Technology | Purpose |
|---|---|
| Ubuntu EC2 | Machine for running Terraform |
| Terraform | Infrastructure automation |
| AWS CLI | Interact with AWS from the terminal |
| IAM | Manage access permissions |
| S3 | Store objects and Terraform remote state |
| EC2 | Create virtual servers |
| VPC | Provide network isolation |
| Security Group | Control inbound and outbound traffic |
| DynamoDB | Demonstrate a database table and legacy state-locking configuration |
| HCL | Terraform configuration language |

## 2. Architecture and Workflow

<pre>
Ubuntu EC2
    |
    +-- Terraform CLI
    |
    +-- AWS CLI / IAM Credentials
              |
              v
          AWS Provider
              |
       +------+-------+
       |      |       |
       v      v       v
      S3     VPC     EC2
       |              |
       |         Security Group
       |
       v
  Remote Terraform State
       |
       +-- S3 state file
       +-- S3 lockfile (state locking)
</pre>

The Terraform workflow is:

```text
Install Tools
     |
Configure AWS Access
     |
Write .tf Files
     |
terraform init
     |
terraform fmt
     |
terraform validate
     |
terraform plan
     |
terraform apply
     |
Verify Resources in AWS
     |
terraform destroy (when appropriate)
```

---

## 3. Install Terraform on Ubuntu EC2

### Step 1: Update the package index

```bash
sudo apt update
```

**Explanation:** Downloads the latest package information from configured Ubuntu repositories.

### Step 2: Install required utilities

```bash
sudo apt install -y curl gnupg unzip
```

**Explanation:**
- `curl`: Downloads files.
- `gnupg`: Handles package-signing keys.
- `unzip`: Extracts ZIP archives.

### Step 3: Add the HashiCorp signing key

```bash
wget -O- https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor \
  -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

**Explanation:**
- `wget -O-`: Sends the downloaded key to standard output.
- `gpg --dearmor`: Converts the ASCII-armored key into a binary keyring.
- `-o`: Specifies the destination file.

### Step 4: Add the Terraform repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(. /etc/os-release && echo "$UBUNTU_CODENAME") main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

**Explanation:**
- `dpkg --print-architecture`: Detects the machine architecture.
- `/etc/os-release`: Provides the Ubuntu release codename.
- `tee`: Writes the repository configuration with elevated permissions.

### Step 5: Install Terraform

```bash
sudo apt update
sudo apt install -y terraform
```

### Step 6: Verify the installation

```bash
terraform -version
```

Expected output format:

```text
Terraform v1.x.x
on linux_amd64
```

Your recorded lab version was Terraform `1.15.8`.

---

## 4. Install and Configure AWS CLI

AWS CLI allows Terraform users to verify AWS access and manage AWS resources from the terminal.

### Step 1: Download AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip"
```

**Important:** The filename is `awscliv2.zip`, not `anscliv2.zip`.

### Step 2: Extract the archive

```bash
unzip awscliv2.zip
```

If the `unzip` command is missing:

```bash
sudo apt update
sudo apt install -y unzip
```

Then run the extraction command again.

### Step 3: Install AWS CLI

```bash
sudo ./aws/install
```

### Step 4: Verify the installation

```bash
aws --version
```

### Step 5: Configure AWS credentials

For an IAM access key:

```bash
aws configure
```

Enter the values interactively:

```text
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_ACCESS_KEY
Default region name: us-east-2
Default output format: json
```

Use replacement placeholders only in documentation. Never paste real credentials into a README or Git repository.

### Step 6: Verify the AWS identity

```bash
aws sts get-caller-identity
```

This command displays the AWS account and identity associated with the active credentials.

### Step 7: List S3 buckets

```bash
aws s3 ls
```

Your terminal showed an existing S3 bucket, confirming that the CLI could list buckets at that point.

### Alternative: AWS CLI login

For supported IAM Identity Center or browser-based authentication workflows, AWS CLI provides interactive login options. For an EC2 instance, an attached IAM role is generally preferable to long-lived access keys.

---

## 5. Create an IAM User and Configure Permissions

Your lab used an IAM user named `udaan-user`.

### Console workflow

1. Open the AWS Management Console.
2. Navigate to IAM.
3. Open **Users**.
4. Create or select `udaan-user`.
5. Assign the permissions required for the lab.
6. If using access keys, open the user's Security credentials.
7. Create an access key for CLI use only if required.
8. Store the credentials securely.

### Permissions used during the lab

| AWS service | Purpose |
|---|---|
| AmazonS3FullAccess | S3 bucket operations |
| AmazonEC2FullAccess | EC2 instance and AMI operations |
| AmazonVPCFullAccess | VPC and security group operations |
| AmazonDynamoDBFullAccess | DynamoDB table operations |

These AWS-managed policies are broad and are suitable only for controlled learning environments when necessary. Prefer least-privilege permissions for real projects.

If Terraform reports an error such as:

```text
not authorized to perform: ec2:DescribeImages
```

check which IAM identity is active and whether its permissions allow the missing action.

For production and long-running environments, prefer IAM roles or another supported temporary-credential mechanism.

---

## 6. Create a Terraform Project

Create a working directory:

```bash
mkdir -p ~/terraform-practice
cd ~/terraform-practice
```

Check the current directory:

```bash
pwd
```

Create the configuration files:

```bash
touch main.tf
touch ec2.tf
touch terraform.tf
```

List the files:

```bash
ls -la
```

Example project structure:

```text
terraform-practice/
├── main.tf
├── ec2.tf
├── terraform.tf
├── .terraform/
├── .terraform.lock.hcl
├── terraform.tfstate
└── terraform.tfstate.backup
```

The last two files are generated when Terraform uses local state. When remote state is configured, the authoritative state is stored in the backend instead.

Do not commit state files or private keys to GitHub.

---

## 7. Understand HCL Syntax

HCL stands for **HashiCorp Configuration Language**.

Terraform configuration files use the `.tf` extension.

### Common Terraform block types

| Block | Purpose |
|---|---|
| `terraform` | Terraform version, provider requirements, backend configuration |
| `provider` | Configures the cloud provider |
| `resource` | Creates or manages infrastructure |
| `data` | Reads existing or externally managed information |
| `variable` | Defines input variables |
| `output` | Displays selected resource information |

### Resource block syntax

```hcl
resource "aws_s3_bucket" "my_s3_bucket" {
  bucket = "your-globally-unique-bucket-name"
}
```

Understanding the syntax:

- `resource`: Declares infrastructure to manage.
- `aws_s3_bucket`: Resource type.
- `my_s3_bucket`: Terraform-local resource name.
- `bucket`: AWS resource argument.
- The value after `=` specifies the bucket name.

The reference to this resource elsewhere in Terraform is:

```hcl
aws_s3_bucket.my_s3_bucket
```

Terraform uses references like this to determine dependencies between resources.

---

## 8. Create an S3 Bucket

Create `main.tf`:

```hcl
resource "aws_s3_bucket" "my_s3_bucket" {
  bucket = "replace-with-your-unique-lowercase-bucket-name"

  tags = {
    Name        = "terraform-practice-bucket"
    Environment = "Lab"
  }
}
```

Replace the placeholder with a globally unique S3 bucket name. Bucket names must follow AWS naming rules, including lowercase letters, numbers, and hyphens where permitted.

For your existing lab, the bucket name was:

```text
anandkumarprajap-bucket
```

Use that name only if it is still available to and controlled by your AWS account.

### Why did the first attempt fail?

The original name included uppercase letters:

```text
Anand-babu-bucket
```

S3 bucket names must be lowercase. Terraform therefore returned a bucket-name validation error.

### Initialize Terraform

```bash
terraform init
```

**Purpose:** Initializes the working directory, installs the required provider plugins, and prepares the configured backend.

### Format the configuration

```bash
terraform fmt
```

### Validate the configuration

```bash
terraform validate
```

### Preview changes

```bash
terraform plan
```

Look for the planned creation of:

```text
aws_s3_bucket.my_s3_bucket
```

### Create the bucket

```bash
terraform apply
```

Review the plan and type:

```text
yes
```

### Verify the bucket

```bash
aws s3 ls
```

You can also inspect the bucket in the S3 section of the AWS Console.

---

## 9. Understand the Terraform Workflow

### `terraform init`

```bash
terraform init
```

Initializes the project and installs providers and modules.

Run it when:
- Starting a new Terraform project.
- Adding or changing providers or modules.
- Changing the backend configuration.

Changing an ordinary resource, data, variable, or output block generally does not require `terraform init`.

### `terraform fmt`

```bash
terraform fmt
```

Formats Terraform configuration files consistently.

### `terraform validate`

```bash
terraform validate
```

Checks Terraform configuration syntax and internal consistency.

### `terraform plan`

```bash
terraform plan
```

Shows the proposed infrastructure changes without applying them.

### `terraform apply`

```bash
terraform apply
```

Previews the plan and requests approval before making changes.

### `terraform destroy`

```bash
terraform destroy
```

Plans the removal of managed infrastructure. Review the plan carefully before approving it.

**Remember:** `plan` previews; `apply` changes infrastructure; `destroy` removes managed infrastructure.

---

## 10. Understand Terraform State

Terraform state maps the resources in your configuration to real infrastructure.

The default local state file is:

```text
terraform.tfstate
```

A previous state backup may be:

```text
terraform.tfstate.backup
```

Inspect the state:

```bash
terraform state list
```

Show a particular resource:

```bash
terraform state show aws_s3_bucket.my_s3_bucket
```

### Why is state important?

Terraform uses state to:
- Track resource IDs and attributes.
- Compare the desired configuration with existing infrastructure.
- Determine what must be created, updated, or destroyed.
- Manage dependencies and resource changes.

### Security warning

Terraform state can contain credentials, sensitive configuration values, and infrastructure details. Do not print or upload the full state publicly.

Add a `.gitignore` file:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
*.pem
*.key
.aws/
awscliv2.zip
```

Do not ignore `.terraform.lock.hcl`; commit it to version control to retain provider selections.

---

## 11. Generate an SSH Key Pair

Run:

```bash
ssh-keygen -t ed25519 -f terra-auto-server-key
```

Choose a passphrase when appropriate.

The generated files are:

```text
terra-auto-server-key
terra-auto-server-key.pub
```

- `terra-auto-server-key`: Private key.
- `terra-auto-server-key.pub`: Public key.

Set restrictive permissions on the private key:

```bash
chmod 600 terra-auto-server-key
```

**Never upload the private key to GitHub.**

### Register the public key with AWS

Add the following to `ec2.tf`:

```hcl
resource "aws_key_pair" "my_ec2_key" {
  key_name   = "terra-auto-server-key"
  public_key = file("${path.module}/terra-auto-server-key.pub")
}
```

Terraform registers the public key in AWS so it can be associated with an EC2 instance.

---

## 12. Create an EC2 Instance Using Terraform

The EC2 configuration uses an AMI data source, the default VPC, a key pair, a security group, and an EC2 instance.

### Step 1: Find an Ubuntu AMI

Add this to `ec2.tf`:

```hcl
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
```

**Explanation:**

- `data`: Reads information rather than creating an AMI.
- `aws_ami`: AWS AMI data-source type.
- `most_recent`: Selects the newest matching image.
- `owners`: Restricts the image search to Canonical's AWS account ID.
- `filter`: Matches the Ubuntu 24.04 image name and HVM virtualization type.

The actual AMI ID is discovered when Terraform queries AWS.

### Step 2: Reference the default VPC

```hcl
data "aws_vpc" "default" {
  default = true
}
```

This reads the existing default VPC rather than attempting to create a new default VPC.

### Step 3: Create a security group

```hcl
resource "aws_security_group" "my_ec2_sg" {
  name        = "terra-auto-sg"
  description = "Allow SSH and HTTP traffic"
  vpc_id      = data.aws_vpc.default.id

  tags = {
    Name = "terra-auto-sg"
  }
}
```

A security group acts as a virtual firewall for associated network interfaces and instances.

### Step 4: Allow SSH inbound traffic

```hcl
resource "aws_vpc_security_group_ingress_rule" "allow_ssh" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "REPLACE_WITH_YOUR_PUBLIC_IP/32"
  from_port         = 22
  to_port           = 22
  ip_protocol       = "tcp"
}
```

Replace the CIDR with your public IP address followed by `/32`.

Avoid `0.0.0.0/0` for SSH in a real deployment because it allows connection attempts from any IPv4 address.

### Step 5: Allow HTTP inbound traffic

For a public web server:

```hcl
resource "aws_vpc_security_group_ingress_rule" "allow_http" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  from_port         = 80
  to_port           = 80
  ip_protocol       = "tcp"
}
```

This permits public IPv4 access to port 80. Add it only if the instance is intended to serve HTTP traffic.

### Step 6: Configure outbound traffic

```hcl
resource "aws_vpc_security_group_egress_rule" "allow_all_traffic_ipv4" {
  security_group_id = aws_security_group.my_ec2_sg.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "-1"
}
```

`-1` means all IP protocols. This permits outbound IPv4 traffic to any destination.

### Step 7: Create the EC2 instance

```hcl
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

**Important arguments:**

| Argument | Explanation |
|---|---|
| `ami` | The Ubuntu image ID |
| `instance_type` | EC2 compute size |
| `key_name` | Registered EC2 key pair |
| `vpc_security_group_ids` | Security groups attached to the instance |
| `volume_size` | Root disk size in GiB |
| `volume_type` | EBS volume type |
| `tags` | Resource metadata |

A `t3.micro` is a small instance type, but verify current regional availability and pricing before deploying.

### Step 8: Validate and deploy

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

Inspect the proposed changes and approve them only if they match your intention.

### Step 9: Connect to the instance

Use the public IP or DNS name shown in the EC2 Console:

```bash
ssh -i terra-auto-server-key ubuntu@YOUR_EC2_PUBLIC_IP
```

The security group must permit SSH from your current public IP. The instance also needs an accessible network path, such as a public IP and suitable subnet routing.

---

## 13. Understand Resource References and Dependencies

Terraform expressions can reference attributes of other resources.

For example:

```hcl
vpc_id = data.aws_vpc.default.id
```

This passes the ID returned by the VPC data source to the security group.

Another example:

```hcl
key_name = aws_key_pair.my_ec2_key.key_name
```

Terraform recognizes the reference and orders the operations so the key pair is available before the instance is created.

This is called an **implicit dependency**.

You generally do not need `depends_on` when Terraform can determine the dependency from a resource reference.

---

## 14. Create Multiple EC2 Instances Using `count`

Terraform supports meta-arguments such as `count` and `for_each`.

To create ten instances, you used:

```hcl
resource "aws_instance" "my_ec2_instance" {
  count = 10

  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t3.micro"
  key_name               = aws_key_pair.my_ec2_key.key_name
  vpc_security_group_ids = [aws_security_group.my_ec2_sg.id]

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }

  tags = {
    Name = "terra-auto-server-${count.index + 1}"
  }
}
```

### How `count` works

- `count = 1`: One instance.
- `count = 3`: Three instances.
- `count = 10`: Ten instances.
- `count.index`: Zero-based index of each instance.

Terraform identifies instances using addresses such as:

```text
aws_instance.my_ec2_instance[0]
aws_instance.my_ec2_instance[1]
aws_instance.my_ec2_instance[2]
```

**Cost warning:** Ten instances and their EBS volumes can incur charges. Start with `count = 1` for practice and scale only when needed.

If you change an existing resource from a single instance to `count = 1`, Terraform may show an address migration from `aws_instance.my_ec2_instance` to `aws_instance.my_ec2_instance[0]`. Review the plan carefully to avoid accidental replacement.

---

## 15. Create a DynamoDB Table

DynamoDB is a managed NoSQL database service. In this lab, it was used to practise table creation and to explore an older Terraform S3 backend locking configuration.

Add the following to `main.tf`:

```hcl
variable "state_table_name" {
  type    = string
  default = "replace-with-your-unique-table-name"
}

resource "aws_dynamodb_table" "my_state_table" {
  name         = var.state_table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = var.state_table_name
  }
}
```

### Explanation

- `variable`: Declares an input value.
- `var.state_table_name`: References the variable.
- `billing_mode = "PAY_PER_REQUEST"`: Uses on-demand capacity billing.
- `hash_key = "LockID"`: Defines the partition key.
- `type = "S"`: Declares a string attribute.

Run:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

If you see:

```text
not authorized to perform: dynamodb:CreateTable
```

check that the active IAM identity has permission to create the table and manage the required DynamoDB operations.

### Important modern Terraform note

The AWS provider and Terraform backend configuration have changed over time. The S3 backend's `dynamodb_table` locking argument is deprecated in current Terraform versions that support S3 native state locking.

For a new setup, use S3 native locking with `use_lockfile = true`. You do not need to create a DynamoDB table solely for Terraform locking when using this approach.

You may still create a DynamoDB table for independent application or learning purposes.

---

## 16. Configure an S3 Remote Backend

A remote backend stores Terraform state outside the machine running Terraform. An S3 backend helps teams share the authoritative state and supports remote state locking.

### Critical design rule

**Create the backend S3 bucket before configuring the project to use it.**

Do not manage the backend bucket using the same Terraform state that is stored inside that bucket. If the state depends on a bucket that the same configuration attempts to destroy, recovery becomes difficult.

Use a separate bootstrap configuration or create the backend bucket independently before configuring the application project's backend.

### Example backend configuration

Create or update `terraform.tf`:

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
    bucket       = "replace-with-your-existing-backend-bucket"
    key          = "terraform-practice/terraform.tfstate"
    region       = "us-east-2"
    use_lockfile = true
  }
}

provider "aws" {
  region = "us-east-2"
}
```

Replace the bucket placeholder with the existing bucket name. The configured bucket must already exist.

### Explanation

| Setting | Purpose |
|---|---|
| `required_version` | Restricts the Terraform CLI version |
| `required_providers` | Declares provider source and version |
| `backend "s3"` | Selects S3 for remote state |
| `bucket` | Existing backend bucket |
| `key` | State file location inside the bucket |
| `region` | AWS region of the bucket |
| `use_lockfile` | Enables S3 native state locking |
| `provider "aws"` | Configures AWS resource operations |

The provider version in this example matches the recorded lab. Keep `.terraform.lock.hcl` committed so the provider selection remains reproducible.

### Initialize the backend

```bash
terraform init
```

If Terraform detects an existing local state file and asks whether to copy it to the new backend, confirm the migration only after checking the target bucket, state contents, and backup plan.

If the backend configuration has changed and Terraform requires explicit migration, use:

```bash
terraform init -migrate-state
```

This attempts to migrate existing state into the newly configured backend.

Use this only when migration is actually intended. Do not use `-reconfigure` as a substitute for migrating state.

### Verify the backend

```bash
terraform state list
```

You can also inspect the expected state location using the AWS CLI:

```bash
aws s3 ls s3://YOUR_BACKEND_BUCKET/terraform-practice/
```

The actual S3 key must match the configured `key` path.

### Permissions required

The identity used by Terraform needs the relevant S3 permissions to read and write the state object and use the lockfile. Depending on the setup, it may also need bucket-listing permissions and access to the encryption key if server-side encryption uses a customer-managed KMS key.

Use a dedicated backend bucket, enable versioning, and restrict access to authorized users and roles.

---

## 17. Understand Backend Migration and Reconfiguration

### Scenario 1: You intentionally want to migrate state

```bash
terraform init -migrate-state
```

Use this when moving an existing state to a new backend and you want Terraform to attempt to copy it.

### Scenario 2: You intentionally want to reconfigure the backend

```bash
terraform init -reconfigure
```

This configures the current working directory for the new backend settings without automatically migrating the previous backend's state.

**Warning:** Reconfiguring is not the same as migrating. Before using it, ensure the correct state already exists at the destination or that you deliberately intend to start with a different state.

### Scenario 3: You changed an ordinary resource block

For an edit such as changing an EC2 instance's `instance_type`, usually run:

```bash
terraform fmt
terraform validate
terraform plan
```

You generally do not need `terraform init` for this change alone.

---

## 18. Terraform State Commands

### List managed resources

```bash
terraform state list
```

Example resource addresses:

```text
aws_s3_bucket.my_s3_bucket
aws_dynamodb_table.my_state_table
aws_key_pair.my_ec2_key
aws_security_group.my_ec2_sg
aws_instance.my_ec2_instance[0]
```

Actual results depend on your configuration and state.

### Inspect a resource

```bash
terraform state show aws_s3_bucket.my_s3_bucket
```

### Show a plan before applying

```bash
terraform plan
```

### Save a plan for review

```bash
terraform plan -out=tfplan
```

Apply the saved plan:

```bash
terraform apply tfplan
```

Plan files can contain sensitive information. Keep them out of version control.

### Refresh and reconcile

Modern Terraform normally refreshes resource information during planning and applying. To inspect the proposed changes, run `terraform plan` instead of relying on an isolated refresh operation.

Avoid using state-manipulation commands unless you understand their effects.

---

## 19. Destroy AWS Resources Safely

To destroy the infrastructure managed by the current Terraform configuration and state:

```bash
terraform plan -destroy
terraform destroy
```

Review the resources Terraform intends to remove before approving the operation.

### Targeted destruction

Terraform supports targeting a specific resource:

```bash
terraform plan -destroy \
  -target=aws_s3_bucket.my_s3_bucket
```

After reviewing the plan, you may execute a targeted destroy if you deliberately need to remove that resource:

```bash
terraform destroy \
  -target=aws_s3_bucket.my_s3_bucket
```

Targeting is an advanced operation and can leave the infrastructure in an intermediate state. Prefer normal configuration changes and a full plan where possible.

### Important backend warning

If the bucket stores the current Terraform state, do **not** destroy it while using that backend.

For your lab, the state bucket and the S3 bucket resource were configured with the same bucket name. Separate the backend bucket from ordinary resources before attempting to destroy or replace it.

A safe sequence is:

1. Identify which bucket stores the state.
2. Back up the state and verify access.
3. Move backend configuration to a separate, durable bucket if necessary.
4. Initialize and verify the new backend.
5. Review the Terraform plan.
6. Destroy only resources that you intentionally want to remove.

Also verify that the EC2 instances, EBS volumes, and other lab resources have been removed when you finish practising.

---

## 20. Common Errors and Solutions

### Error 1: `unzip: command not found`

Install unzip:

```bash
sudo apt update
sudo apt install -y unzip
```

Then retry:

```bash
unzip awscliv2.zip
```

### Error 2: Invalid S3 bucket name

Example:

```text
only lowercase alphanumeric characters and hyphens allowed
```

**Cause:** The bucket name contains uppercase characters or otherwise violates S3 naming rules.

**Solution:** Use a valid, globally unique lowercase bucket name.

### Error 3: `ec2:DescribeImages` is not authorized

**Cause:** The active IAM identity does not have permission to query AMIs.

**Solution:**

```bash
aws sts get-caller-identity
```

Verify the identity, then update its IAM permissions according to least privilege.

### Error 4: `dynamodb:CreateTable` is not authorized

**Cause:** The IAM identity cannot create the DynamoDB table.

**Solution:** Grant the required DynamoDB permissions to the correct identity and retry.

### Error 5: Backend configuration changed

Example:

```text
Error: Backend configuration changed
```

**Solution:** Decide whether the old state should be migrated or the current backend should simply be reconfigured.

Migrate state:

```bash
terraform init -migrate-state
```

Reconfigure without automatic migration:

```bash
terraform init -reconfigure
```

Choose the command based on the intended state transition.

### Error 6: Deprecated `dynamodb_table` backend parameter

**Cause:** The configuration uses the older S3 backend locking option.

**Solution:** For a new setup, use:

```hcl
use_lockfile = true
```

Ensure the Terraform version and permissions support S3 native state locking.

### Error 7: `terraform plan` shows unexpected changes

**Solution:**

```bash
terraform state list
terraform plan
```

Review the resource addresses, configuration, provider settings, and backend state. Do not approve a plan you do not understand.

---

## 21. Best Practices

- Keep Terraform configurations in version control.
- Commit `.terraform.lock.hcl`.
- Never commit real AWS access keys, private SSH keys, or state files.
- Prefer IAM roles and temporary credentials over long-lived access keys.
- Use least-privilege IAM permissions.
- Use a dedicated S3 backend bucket, with versioning and restricted access.
- Use `use_lockfile = true` for supported S3 backend state locking.
- Review `terraform plan` before every important apply or destroy.
- Restrict SSH access to trusted IP addresses.
- Begin EC2 practice with one instance rather than ten.
- Use tags to identify lab resources.
- Check AWS costs and clean up resources after the lab.
- Do not destroy the bucket containing the active Terraform state.
- Run `terraform init` when backend, provider, or module initialization requirements change—not after every resource edit.

---

## 22. Complete Command Reference

| Command | Purpose |
|---|---|
| `sudo apt update` | Refresh package metadata |
| `terraform -version` | Check Terraform version |
| `aws --version` | Check AWS CLI version |
| `aws configure` | Configure access keys and defaults |
| `aws sts get-caller-identity` | Verify AWS identity |
| `aws s3 ls` | List S3 buckets |
| `terraform init` | Initialize project/backend/providers |
| `terraform fmt` | Format `.tf` files |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply approved changes |
| `terraform state list` | List resources in state |
| `terraform state show ADDRESS` | Inspect a resource |
| `terraform init -migrate-state` | Attempt state migration |
| `terraform init -reconfigure` | Reconfigure backend without automatic migration |
| `terraform plan -destroy` | Preview resource destruction |
| `terraform destroy` | Destroy managed infrastructure |

---

## Final Learning Outcome

After completing this hands-on practice, you should understand how to:

1. Install Terraform and AWS CLI on Ubuntu.
2. Configure AWS access and troubleshoot IAM permissions.
3. Write HCL configuration files.
4. Create an S3 bucket.
5. Understand resources, data sources, variables, and references.
6. Generate an SSH key pair.
7. Provision an EC2 instance with a security group and EBS root volume.
8. Create multiple instances using `count`.
9. Create a DynamoDB table.
10. Configure S3 remote state and native state locking.
11. Understand state migration and backend reconfiguration.
12. Plan, apply, inspect, and safely destroy infrastructure.

**Recommended execution sequence:**

```bash
cd ~/terraform-practice

terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform state list
```

Review all planned changes and verify the created resources in AWS. Clean up only the lab resources you no longer need, preserving any remote state backend that is still in use.
