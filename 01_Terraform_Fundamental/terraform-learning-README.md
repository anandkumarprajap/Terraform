# 🏗️ Terraform — Infrastructure as Code (Beginner to Advanced Guide)

> **Terraform** is a declarative **Infrastructure as Code (IaC)** tool used to provision, change, and manage infrastructure through configuration files.
>
> **Important:** Terraform uses **HCL (HashiCorp Configuration Language)**. It is not "SCL". The company is **HashiCorp**, not "Hossi Corp".

---

# 📚 Table of Contents

1. What is Terraform?
2. What is Infrastructure as Code?
3. Why IaC Matters
4. Declarative vs Imperative
5. Terraform in Modern DevOps
6. Terraform Architecture
7. Terraform Lifecycle and Workflow
8. Terraform Providers
9. Terraform Resources
10. Terraform HCL Basics
11. Blocks, Arguments, Attributes and Expressions
12. Terraform Blocks
13. Variables
14. Outputs
15. Locals
16. Data Sources
17. Comments and Formatting
18. Terraform State
19. Dependency Graph
20. Terraform vs Ansible
21. Terraform vs CloudFormation
22. Terraform vs OpenTofu
23. Terraform vs Pulumi
24. Terraform vs Crossplane
25. Infrastructure as Code Tools
26. Cloud Providers and Hyperscalers
27. Terraform History
28. HashiCorp Licensing and OpenTofu
29. HashiCorp Product Ecosystem
30. Installation Overview
31. Windows Installation
32. macOS Installation
33. Linux Installation
34. AWS EC2 Ubuntu Installation
35. AWS CLI Installation
36. AWS Credentials
37. Recommended Folder Structure
38. AWS EC2 Practical Project
39. Terraform Commands
40. Complete EC2 Example
41. Destroy Infrastructure
42. Common Errors
43. Security Best Practices
44. Terraform with Git and CI/CD
45. Terraform + Ansible
46. Production Workflow
47. Interview Revision
48. Complete Learning Roadmap
49. Key Points
50. Official Documentation

---

# 🏗️ What is Terraform?

Terraform is an **Infrastructure as Code** tool created by **HashiCorp**.

Instead of manually creating infrastructure from a cloud console, we write configuration files that describe the infrastructure we want.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"
}
```

Then Terraform can:

```bash
terraform init
terraform plan
terraform apply
```

Terraform communicates with cloud/provider APIs and creates the infrastructure.

---

# 🧠 Easy Terraform Example

Imagine you want an EC2 server.

## Manual method

You open AWS Console:

```text
AWS Console
     ↓
EC2
     ↓
Launch Instance
     ↓
Select AMI
     ↓
Select Instance Type
     ↓
Select Key Pair
     ↓
Configure Network
     ↓
Configure Security Group
     ↓
Launch
```

## Terraform method

You write:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"
}
```

Then:

```bash
terraform init
terraform plan
terraform apply
```

Terraform performs the required API operations.

---

# 🏗️ What is Infrastructure as Code?

**Infrastructure as Code (IaC)** means managing infrastructure using machine-readable configuration files instead of depending primarily on manual cloud-console operations.

Infrastructure can include:

```text
VPC
Subnet
Route Table
Internet Gateway
NAT Gateway
Security Group
EC2
Load Balancer
RDS
S3
IAM
DNS
Kubernetes
Monitoring
```

---

# 🔥 Why Infrastructure as Code Matters

Without IaC:

```text
Engineer
   ↓
Cloud Console
   ↓
Manual Configuration
   ↓
Human Errors
   ↓
Different Environments
```

With IaC:

```text
Developer
    ↓
Git Repository
    ↓
Terraform Code
    ↓
terraform plan
    ↓
Code Review
    ↓
terraform apply
    ↓
Cloud Infrastructure
```

IaC provides:

- Automation
- Repeatability
- Version control
- Consistency
- Reproducibility
- Code review
- Auditability
- Faster provisioning
- Easier disaster recovery
- Easier environment creation

---

# 🧑‍💻 Infrastructure as Code Example

Suppose a company needs:

```text
Production Environment

VPC
 ├── Public Subnet
 ├── Private Subnet
 ├── EC2
 ├── Load Balancer
 └── Database
```

Instead of manually creating everything, Terraform code can describe it:

```text
Terraform Code
      ↓
AWS Provider
      ↓
AWS API
      ↓
VPC
      ↓
Subnets
      ↓
EC2
      ↓
Load Balancer
      ↓
Database
```

The same design can be reused for:

```text
Development
Staging
Production
```

---

# 🔄 Declarative vs Imperative

Terraform is primarily **declarative**.

---

## 🔨 Imperative Approach

Imperative means:

> Tell the system **how** to perform the work.

Example:

```text
1. Create VPC
2. Create subnet
3. Create route table
4. Create Internet Gateway
5. Create security group
6. Create EC2
7. Attach security group
8. Configure networking
```

A manual AWS Console workflow is experienced in a similar step-by-step way:

```text
AWS Console
    ↓
EC2
    ↓
Select AMI
    ↓
Select Instance Type
    ↓
Select SSH Key
    ↓
Configure Network
    ↓
Configure Security Group
    ↓
Launch Instance
```

---

## 📜 Declarative Approach

Declarative means:

> Tell the system **what desired state should exist**.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"
}
```

You are saying:

```text
I want an EC2 instance
with this AMI
and this instance type.
```

You are not manually writing every AWS API operation.

Terraform determines what actions are required.

---

# 🆚 Imperative vs Declarative

| Imperative | Declarative |
|---|---|
| Tells how | Tells what |
| Step-by-step | Desired state |
| Procedural | State-oriented |
| User controls operations | Tool determines required changes |
| Example: shell script | Example: Terraform |

---

# ☸️ Terraform and Desired State

Terraform follows the idea:

```text
Desired State
     vs
Known/Current State
     ↓
Calculate Changes
     ↓
Apply Changes
```

Example:

```text
Desired:
3 EC2 instances

Current:
2 EC2 instances

Terraform:
Create 1 more EC2 instance
```

---

# 🚀 Terraform in Modern DevOps

Terraform commonly acts as the **infrastructure provisioning layer** in DevOps.

```text
Developer
    ↓
Git
    ↓
CI/CD
    ↓
Terraform
    ↓
Cloud Infrastructure
    ↓
Kubernetes / Servers
    ↓
Application
```

Terraform can create:

```text
VPC
EC2
EKS
RDS
S3
IAM
Load Balancer
Route 53
CloudWatch
Security Groups
```

---

# 🏢 Company Example

Think of a company:

| Company | Terraform |
|---|---|
| Infrastructure team | Terraform |
| Infrastructure blueprint | `.tf` files |
| Cloud provider | AWS/Azure/GCP |
| Infrastructure object | Resource |
| Configuration input | Variable |
| Result | Output |
| Infrastructure memory | State |
| External connection | Provider |

Terraform is like the **infrastructure architect and automation engine**.

---

# 🏛️ Terraform Architecture

```text
+------------------------------------------------------+
|                  Terraform Project                   |
|                                                      |
|  main.tf   variables.tf   outputs.tf   providers.tf |
+---------------------------+--------------------------+
                            |
                            ▼
                    Terraform CLI
                            |
                            ▼
                       Provider
                            |
                            ▼
                       Provider API
                            |
            +---------------+---------------+
            |               |               |
            ▼               ▼               ▼
           AWS            Azure            GCP
            |
            ▼
       Cloud Resources
```

---

# 🔌 Terraform Provider

A **provider** is a plugin that allows Terraform to communicate with an external platform or service.

Examples:

```text
AWS
Azure
Google Cloud
Kubernetes
GitHub
Cloudflare
Vault
```

AWS example:

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

Think of a provider as a **translator**:

```text
Terraform
    ↓
AWS Provider
    ↓
AWS API
```

---

# 🧱 Terraform Resource

A **resource** represents infrastructure that Terraform manages.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"
}
```

Syntax:

```hcl
resource "<RESOURCE_TYPE>" "<LOCAL_NAME>" {
  arguments
}
```

Here:

```text
Resource type = aws_instance
Local name    = web
```

Reference:

```hcl
aws_instance.web.id
```

---

# 📝 Terraform HCL

Terraform configuration normally uses:

**HCL — HashiCorp Configuration Language**

Terraform files:

```text
.tf
```

Examples:

```text
main.tf
providers.tf
variables.tf
outputs.tf
locals.tf
data.tf
versions.tf
```

Terraform can also use JSON-style Terraform configuration:

```text
.tf.json
```

But HCL is the common human-readable format.

---

# 🧩 Blocks

A block is a major building unit of Terraform configuration.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"
}
```

This contains:

```text
Block Type
   ↓
resource

Labels
   ↓
aws_instance
web

Body
   ↓
arguments
```

---

# 📌 Arguments

An argument assigns a value.

Example:

```hcl
instance_type = "t3.micro"
```

Another:

```hcl
region = "ap-south-1"
```

---

# 📤 Attributes

An attribute is a value exposed by a resource or data object that can be referenced.

Example:

```hcl
aws_instance.web.id
```

Here:

```text
aws_instance.web
        ↓
      resource
        ↓
        id
        ↓
    instance ID
```

Important distinction:

```text
Argument
= value you configure

Attribute
= value exposed by a resource/data object
```

---

# 🧮 Expressions

Expressions calculate or reference values.

Example:

```hcl
instance_type = var.instance_type
```

Here:

```text
var.instance_type
```

is an expression.

Another:

```hcl
name = "${var.project_name}-${var.environment}"
```

---

# 🧱 Important Terraform Blocks

The major blocks you should learn are:

```text
terraform
provider
resource
variable
output
locals
data
module
```

Example:

```hcl
terraform {
  required_version = ">= 1.5.0"
}

provider "aws" {
  region = "ap-south-1"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

locals {
  project_name = "terraform-demo"
}

data "aws_caller_identity" "current" {}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
}

output "instance_id" {
  value = aws_instance.web.id
}
```

---

# 🔄 Terraform Lifecycle and Workflow

The basic Terraform workflow is:

```text
Write Code
    ↓
terraform init
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform plan
    ↓
terraform apply
    ↓
Infrastructure
    ↓
terraform destroy
```

---

# 1️⃣ terraform init

Command:

```bash
terraform init
```

Purpose:

```text
Initialize working directory
        ↓
Download providers
        ↓
Initialize modules
        ↓
Configure backend
        ↓
Prepare Terraform
```

Typical output includes:

```text
Initializing the backend...
Initializing provider plugins...
Terraform has been successfully initialized!
```

---

# 2️⃣ terraform fmt

Command:

```bash
terraform fmt
```

Purpose:

```text
Format Terraform code
```

Check formatting without changing files:

```bash
terraform fmt -check
```

Format recursively:

```bash
terraform fmt -recursive
```

---

# 3️⃣ terraform validate

Command:

```bash
terraform validate
```

Purpose:

```text
Check configuration syntax
        ↓
Check internal consistency
        ↓
Report configuration errors
```

---

# 4️⃣ terraform plan

Command:

```bash
terraform plan
```

Purpose:

```text
Read configuration
      ↓
Read state
      ↓
Compare desired/current information
      ↓
Calculate changes
      ↓
Show proposed actions
```

Common symbols:

```text
+   create

~   update

-   destroy

-/+ replace
```

Example:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

---

# 5️⃣ terraform apply

Command:

```bash
terraform apply
```

Terraform displays the plan and asks for confirmation.

```text
Do you want to perform these actions?
```

Enter:

```text
yes
```

Terraform then calls the provider APIs.

For automation:

```bash
terraform apply -auto-approve
```

Use this carefully, especially in production.

---

# 6️⃣ terraform destroy

Command:

```bash
terraform destroy
```

Terraform plans removal of managed resources.

```text
Infrastructure
      ↓
terraform destroy
      ↓
Cloud resources removed
```

This is especially useful for temporary learning environments.

---

# 🔁 Complete Terraform Request Flow

```text
Developer
    │
    ▼
Terraform Code
    │
    ▼
terraform init
    │
    ▼
Provider
    │
    ▼
terraform plan
    │
    ▼
Review Plan
    │
    ▼
terraform apply
    │
    ▼
Provider API
    │
    ▼
AWS / Azure / GCP
    │
    ▼
Infrastructure Created
```

---

# 📦 Terraform Variables

Variables are inputs to Terraform modules.

Example:

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
```

Use it:

```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

---

# 🔢 Variable Types

Common Terraform types:

```text
string
number
bool
list
set
map
object
tuple
```

String:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}
```

Number:

```hcl
variable "server_count" {
  type    = number
  default = 2
}
```

Boolean:

```hcl
variable "enable_monitoring" {
  type    = bool
  default = true
}
```

List:

```hcl
variable "availability_zones" {
  type = list(string)

  default = [
    "ap-south-1a",
    "ap-south-1b"
  ]
}
```

Map:

```hcl
variable "tags" {
  type = map(string)

  default = {
    Environment = "dev"
    Project     = "terraform"
  }
}
```

---

# 📤 Terraform Outputs

Outputs expose useful information.

Example:

```hcl
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}
```

Public IP:

```hcl
output "public_ip" {
  description = "EC2 public IP"
  value       = aws_instance.web.public_ip
}
```

Run:

```bash
terraform output
```

Specific output:

```bash
terraform output public_ip
```

Sensitive output:

```hcl
output "secret_value" {
  value     = var.secret_value
  sensitive = true
}
```

Remember:

```text
sensitive = true
```

hides normal CLI display but does not automatically remove the value from Terraform state.

---

# 🏠 Terraform Locals

`locals` create reusable local expressions.

Example:

```hcl
locals {
  project     = "terraform-demo"
  environment = "dev"

  common_tags = {
    Project     = "terraform-demo"
    Environment = "dev"
  }
}
```

Use:

```hcl
tags = local.common_tags
```

Another example:

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}
```

Use:

```hcl
name = "${local.name_prefix}-server"
```

---

# 🔎 Terraform Data Sources

A `data` block reads existing information.

Example:

```hcl
data "aws_caller_identity" "current" {}
```

Use:

```hcl
data.aws_caller_identity.current.account_id
```

Think:

```text
resource
   ↓
Create/manage something

data
   ↓
Read information about something
```

---

# 🖼️ Data Source Example — AMI

AMI IDs are region-specific.

A data source can discover an appropriate AMI:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]

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

Then:

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
}
```

Always verify AMI filters and owner IDs for the region and OS version you are using.

---

# 💬 Terraform Comments

Single-line:

```hcl
# This creates an EC2 instance
```

Also:

```hcl
// This creates an EC2 instance
```

Multi-line:

```hcl
/*
  This is a
  multi-line comment.
*/
```

Recommended:

```hcl
# Create EC2 web server
resource "aws_instance" "web" {
  ...
}
```

---

# 🧹 Terraform Formatting

Use:

```bash
terraform fmt
```

Check:

```bash
terraform fmt -check
```

Recursive:

```bash
terraform fmt -recursive
```

---

# 💾 Terraform State

Terraform uses **state** to keep track of infrastructure it manages.

Default local state:

```text
terraform.tfstate
```

Concept:

```text
Terraform Configuration
        ↕
Terraform State
        ↕
Real Infrastructure
```

State can contain:

```text
Resource IDs
Attributes
Dependencies
Known values
Sensitive values
Infrastructure metadata
```

---

# 🚨 Why Terraform State is Important

Suppose:

```text
Terraform Code
     ↓
EC2 instance
     ↓
i-123456789
```

Terraform needs to know:

```text
Which resource in AWS
belongs to which Terraform resource?
```

State helps maintain this mapping.

---

# 🔐 Do Not Commit State to Git

Do not normally commit:

```text
terraform.tfstate
terraform.tfstate.backup
```

Recommended `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json
crash.log
crash.*.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```

The provider lock file should normally be committed:

```text
.terraform.lock.hcl
```

---

# 🔗 Terraform Dependency Graph

Example:

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
}

resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.web.id]
}
```

Terraform understands:

```text
Security Group
      ↓
EC2 Instance
```

The EC2 instance depends on the security group.

Terraform can therefore determine the required ordering.

---

# 🆚 Terraform vs Ansible

Terraform:

```text
Infrastructure Provisioning
```

Ansible:

```text
Configuration Management
```

---

## Terraform

Terraform commonly manages:

```text
VPC
Subnet
EC2
S3
RDS
IAM
Load Balancer
DNS
Kubernetes infrastructure
```

Language:

```text
HCL
```

Model:

```text
Declarative
```

---

## Ansible

Ansible commonly manages:

```text
Packages
Users
Files
Services
Applications
Server configuration
```

Language:

```text
YAML
```

Example:

```yaml
- name: Install nginx
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

---

# 🧠 Easy Terraform vs Ansible

Remember:

```text
Terraform
"What infrastructure should exist?"

Ansible
"How should the existing machines be configured?"
```

Typical combination:

```text
Terraform
    ↓
VPC
    ↓
EC2
    ↓
Security Group
    ↓
Load Balancer
    ↓
Ansible
    ↓
Install Docker
    ↓
Install Nginx
    ↓
Configure Application
```

Ansible can also manage cloud resources, but Terraform's primary design is infrastructure lifecycle management using state.

---

# 🆚 Terraform vs AWS CloudFormation

CloudFormation is AWS's native Infrastructure as Code service.

| Feature | Terraform | CloudFormation |
|---|---|---|
| Main vendor | HashiCorp / IBM context | AWS |
| Language | HCL | YAML / JSON |
| AWS | Yes | Yes |
| Azure | Yes | No |
| GCP | Yes | No |
| Multi-cloud | Strong | AWS-focused |
| AWS-native | Strong | Excellent |
| Provider ecosystem | Large | AWS services |

CloudFormation can be a strong choice for AWS-only environments.

Terraform is often attractive for multi-provider environments.

---

# 🆚 Terraform vs OpenTofu

OpenTofu is an open-source fork of Terraform created after HashiCorp changed Terraform's licensing model in 2023.

Both have concepts such as:

```text
HCL-style configuration
Providers
Resources
Modules
State
Plan
Apply
Destroy
```

Simple comparison:

```text
Terraform
    ↓
HashiCorp product
    ↓
Newer releases use BUSL/BSL 1.1

OpenTofu
    ↓
Open-source fork
    ↓
Linux Foundation stewardship
```

Do not assume every Terraform and OpenTofu version is perfectly interchangeable.

Check:

```text
CLI version
Provider compatibility
Module compatibility
State compatibility
Backend behavior
Team standards
Licensing requirements
```

---

# 🆚 Terraform vs Pulumi

Terraform primarily uses HCL.

Pulumi allows infrastructure to be defined using general-purpose programming languages.

Common Pulumi languages:

```text
TypeScript
JavaScript
Python
Go
C#
Java
YAML
```

Terraform:

```hcl
resource "aws_instance" "web" {
  instance_type = "t3.micro"
  ami           = var.ami_id
}
```

Pulumi can use Python:

```python
import pulumi_aws as aws

server = aws.ec2.Instance(
    "web",
    instance_type="t3.micro",
    ami="ami-xxxxxxxxxxxxxxxxx"
)
```

Simple idea:

```text
Terraform → HCL-focused declarative IaC

Pulumi → General-purpose programming languages for IaC
```

---

# 🆚 Terraform vs Crossplane

Crossplane is Kubernetes-oriented infrastructure management.

Terraform:

```text
Terraform CLI
      ↓
Providers
      ↓
Cloud APIs
```

Crossplane:

```text
Kubernetes API
      ↓
Crossplane
      ↓
Cloud APIs
```

Crossplane is useful when infrastructure management should be deeply integrated with a Kubernetes control plane.

---

# 🛠️ Infrastructure as Code Tools

Common infrastructure tools:

| Tool | Primary Model | Typical Use |
|---|---|---|
| Terraform | Declarative | Multi-provider IaC |
| OpenTofu | Declarative | Open-source Terraform-compatible IaC |
| CloudFormation | Declarative | AWS-native IaC |
| Pulumi | Declarative + programming languages | Multi-cloud IaC |
| Ansible | Declarative automation | Configuration management |
| Crossplane | Kubernetes control plane | Kubernetes-centric infrastructure |
| AWS CDK | Programming-language abstractions | AWS IaC |

Important:

```text
All are not identical.

Different tools solve different infrastructure problems.
```

---

# ☁️ Cloud Providers and Hyperscalers

Major cloud platforms include:

```text
AWS
Microsoft Azure
Google Cloud
Alibaba Cloud
```

---

# 🟧 AWS

Common services:

```text
EC2
VPC
S3
RDS
EKS
IAM
Lambda
CloudFront
Route 53
ELB
```

---

# 🔵 Microsoft Azure

Examples:

```text
Virtual Machines
Virtual Network
AKS
Azure Storage
Azure SQL
```

---

# 🔴 Google Cloud

Examples:

```text
Compute Engine
VPC
GKE
Cloud Storage
Cloud SQL
```

---

# 🟠 Alibaba Cloud

Examples:

```text
ECS
VPC
OSS
RDS
ACK
```

Terraform can use provider plugins to communicate with many cloud platforms.

---

# 📜 Terraform History

Terraform was created by **HashiCorp**.

Terraform 0.1 was released in **July 2014**.

Many training materials identify:

```text
18 July 2014
```

as the launch date.

For interviews, the safest wording is:

```text
Terraform 0.1 was released in July 2014.
```

Terraform began as an open-source, cloud-agnostic IaC tool and later developed a large provider ecosystem.

---

# 🏢 HashiCorp

HashiCorp is the company behind Terraform and several infrastructure/security technologies.

Important products include:

```text
Terraform
Vagrant
Vault
Boundary
Packer
Consul
Nomad
```

---

# 📦 Vagrant

Vagrant focuses on development environments.

```text
Developer
    ↓
Vagrant
    ↓
Local Development Environment
    ↓
Virtual Machine
```

---

# 🔐 Vault

Vault focuses on:

```text
Secrets
Credentials
Tokens
Identity
Encryption
```

---

# 🚪 Boundary

Boundary focuses on secure access to remote infrastructure.

Conceptually:

```text
User
  ↓
Boundary
  ↓
Target
```

---

# 🏢 HashiCorp and IBM

IBM announced its acquisition of HashiCorp in 2024.

The acquisition was completed on:

```text
February 27, 2025
```

So:

```text
2024 → Acquisition announced
2025 → Acquisition completed
```

---

# 📜 Terraform Licensing Change

A common interview question is:

> Why is Terraform not open source in the newer generation?

The precise answer is:

```text
Older Terraform releases
        ↓
MPL 2.0

August 2023
        ↓
HashiCorp changes future product licensing

Newer Terraform releases
        ↓
Business Source License 1.1
(BSL/BUSL)
```

The Business Source License is generally described as **source-available**, not an OSI-approved open-source license.

Do not simply say:

```text
Terraform became closed source.
```

A better answer is:

```text
Newer Terraform releases moved from MPL 2.0
to the Business Source License 1.1.

This is source-available rather than an
OSI-approved open-source license.
```

HashiCorp stated that the change was intended to protect its ability to invest in its products and prevent competitors from taking the source code and creating competing commercial offerings.

Always check the exact license for the Terraform version you use.

---

# 🌱 Why OpenTofu Was Created

After HashiCorp announced the licensing change in August 2023, members of the infrastructure community created OpenTofu.

Timeline:

```text
2014
  ↓
Terraform 0.1

       ↓

MPL 2.0 era

       ↓

August 2023
HashiCorp announces BSL

       ↓

August 2023
OpenTofu announced

       ↓

September 5, 2023
OpenTofu public fork
```

OpenTofu goals include:

```text
Open-source governance
Vendor neutrality
Community development
Terraform workflow compatibility
Open ecosystem
```

OpenTofu is under Linux Foundation stewardship.

---

# 💻 Terraform Installation

You can install Terraform on:

```text
Windows
macOS
Linux
AWS EC2
```

Always verify your CPU architecture and install the appropriate package.

---

# 🪟 Windows Installation

Use the official Terraform installation documentation.

```text
https://developer.hashicorp.com/terraform/install
```

Download the Windows binary matching your architecture.

Extract:

```text
terraform.exe
```

For example:

```text
C:\Terraform\terraform.exe
```

Add:

```text
C:\Terraform
```

to the Windows `PATH`.

Open a new PowerShell terminal.

Verify:

```powershell
terraform --version
```

Check executable:

```powershell
Get-Command terraform
```

Help:

```powershell
terraform -help
```

---

# 🪟 Windows — Chocolatey

If Chocolatey is installed:

```powershell
choco install terraform
```

Verify:

```powershell
terraform --version
```

The Chocolatey package is maintained through the Chocolatey ecosystem, not directly by HashiCorp.

---

# 🍎 macOS Installation

Using Homebrew:

```bash
brew tap hashicorp/tap
```

Then:

```bash
brew install hashicorp/tap/terraform
```

Verify:

```bash
terraform --version
```

Find Terraform:

```bash
which terraform
```

Update:

```bash
brew update
brew upgrade hashicorp/tap/terraform
```

Important:

```text
HashiCorp
```

Correct spelling.

Not:

```text
Hossi Corp
```

---

# 🐧 Linux Installation

Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y gnupg software-properties-common
```

Add HashiCorp GPG key:

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```

Add repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) \
signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com \
$(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Update:

```bash
sudo apt-get update
```

Install:

```bash
sudo apt-get install terraform
```

Verify:

```bash
terraform --version
```

---

# ☁️ Terraform Installation on AWS EC2

Suppose you have an Ubuntu EC2 instance.

Connect:

```bash
ssh -i "your-key.pem" ubuntu@YOUR_PUBLIC_IP
```

Update:

```bash
sudo apt-get update
```

Install prerequisites:

```bash
sudo apt-get install -y wget gnupg software-properties-common unzip
```

Add HashiCorp key:

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```

Add repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) \
signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com \
$(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Update:

```bash
sudo apt-get update
```

Install:

```bash
sudo apt-get install terraform
```

Verify:

```bash
terraform --version
```

---

# 🔑 AWS CLI

Terraform needs a way to authenticate to AWS.

Install AWS CLI first.

Verify:

```bash
aws --version
```

Then:

```bash
aws configure
```

You will be asked for:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

Example:

```text
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_KEY
Default region name [None]: ap-south-1
Default output format [None]: json
```

Never put real credentials in Git.

---

# 🧪 Verify AWS Credentials

Run:

```bash
aws sts get-caller-identity
```

Example:

```json
{
  "UserId": "AIDAXXXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/terraform-user"
}
```

If this command works, your AWS CLI authentication is working.

---

# 👤 AWS Profiles

Create a named profile:

```bash
aws configure --profile terraform-dev
```

Verify:

```bash
aws sts get-caller-identity --profile terraform-dev
```

Terraform provider:

```hcl
provider "aws" {
  region  = "ap-south-1"
  profile = "terraform-dev"
}
```

For production, consider:

```text
IAM roles
IAM Identity Center
Short-lived credentials
CI/CD OIDC
EC2 instance profiles
```

instead of long-lived access keys.

---

# 🔐 AWS Credentials Locations

Linux/macOS:

```text
~/.aws/credentials
~/.aws/config
```

Windows:

```text
C:\Users\USERNAME\.aws\credentials
C:\Users\USERNAME\.aws\config
```

---

# 📁 Recommended Terraform Folder Structure

Simple project:

```text
terraform-aws-ec2/
│
├── README.md
├── .gitignore
├── versions.tf
├── providers.tf
├── main.tf
├── variables.tf
├── outputs.tf
├── locals.tf
├── data.tf
└── terraform.tfvars
```

Advanced project:

```text
terraform-project/
│
├── README.md
├── .gitignore
├── versions.tf
├── providers.tf
├── main.tf
├── variables.tf
├── outputs.tf
├── locals.tf
├── data.tf
├── terraform.tfvars.example
│
├── modules/
│   ├── network/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   └── ec2/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

---

# 🗂️ What Each File Does

```text
versions.tf
    ↓
Terraform/provider version requirements

providers.tf
    ↓
Provider configuration

main.tf
    ↓
Main resources

variables.tf
    ↓
Input variables

outputs.tf
    ↓
Useful outputs

locals.tf
    ↓
Reusable local values

data.tf
    ↓
Data sources

terraform.tfvars
    ↓
Variable values

README.md
    ↓
Project documentation
```

---

# 🚀 AWS EC2 Terraform Project

Goal:

```text
Create an EC2 instance using Terraform
```

Architecture:

```text
Laptop / EC2
     ↓
Terraform CLI
     ↓
AWS Provider
     ↓
AWS API
     ↓
EC2
```

Requirements:

```text
AWS Account
AWS CLI
Terraform
AWS Credentials
Existing EC2 Key Pair
Valid AMI
```

Verify:

```bash
terraform --version
```

```bash
aws --version
```

```bash
aws sts get-caller-identity
```

---

# 📄 Step 1 — Create Project Directory

Linux/macOS:

```bash
mkdir terraform-ec2
cd terraform-ec2
```

Windows PowerShell:

```powershell
mkdir terraform-ec2
cd terraform-ec2
```

---

# 📄 Step 2 — versions.tf

Create:

```text
versions.tf
```

Code:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}
```

---

# 📄 Step 3 — providers.tf

Create:

```text
providers.tf
```

Code:

```hcl
provider "aws" {
  region = var.aws_region
}
```

---

# 📄 Step 4 — variables.tf

Create:

```text
variables.tf
```

Code:

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "ap-south-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "ami_id" {
  description = "AMI ID valid in the selected AWS region"
  type        = string
}

variable "key_name" {
  description = "Existing EC2 key pair name"
  type        = string
}
```

---

# 📄 Step 5 — main.tf

Create:

```text
main.tf
```

Code:

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = {
    Name        = "terraform-web"
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}
```

---

# 📄 Step 6 — outputs.tf

Create:

```text
outputs.tf
```

Code:

```hcl
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "EC2 public IP"
  value       = aws_instance.web.public_ip
}

output "public_dns" {
  description = "EC2 public DNS"
  value       = aws_instance.web.public_dns
}
```

---

# 📄 Step 7 — terraform.tfvars

Create:

```text
terraform.tfvars
```

Example:

```hcl
aws_region    = "ap-south-1"
instance_type = "t3.micro"
ami_id        = "ami-xxxxxxxxxxxxxxxxx"
key_name      = "your-existing-key-pair"
```

Important:

```text
AMI ID is region-specific.
```

Do not copy an AMI ID from another region without checking it.

---

# 🧪 Step 8 — terraform init

Run:

```bash
terraform init
```

Expected process:

```text
Terraform Project
      ↓
terraform init
      ↓
Download AWS Provider
      ↓
Initialize Terraform
```

---

# 🧹 Step 9 — terraform fmt

Run:

```bash
terraform fmt
```

---

# ✅ Step 10 — terraform validate

Run:

```bash
terraform validate
```

Expected:

```text
Success! The configuration is valid.
```

---

# 📋 Step 11 — terraform plan

Run:

```bash
terraform plan
```

Review carefully.

Expected:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

---

# 🚀 Step 12 — terraform apply

Run:

```bash
terraform apply
```

Terraform asks:

```text
Do you want to perform these actions?
```

Enter:

```text
yes
```

Terraform creates the EC2 instance.

---

# 📤 Step 13 — Terraform Output

Run:

```bash
terraform output
```

Example:

```text
instance_id = "i-xxxxxxxxxxxxxxxxx"
public_ip   = "13.x.x.x"
public_dns  = "ec2-13-x-x-x.ap-south-1.compute.amazonaws.com"
```

Get only the public IP:

```bash
terraform output public_ip
```

---

# 🔍 Inspect Terraform State

List managed resources:

```bash
terraform state list
```

Example:

```text
aws_instance.web
```

Show a resource:

```bash
terraform state show aws_instance.web
```

Show complete state information:

```bash
terraform show
```

---

# 💥 Destroy the EC2 Instance

When the learning project is complete:

```bash
terraform destroy
```

Confirm:

```text
yes
```

Then verify in AWS Console.

This helps avoid unnecessary cloud charges.

---

# 🧪 Complete Project Workflow

```text
Create Directory
      ↓
Create .tf files
      ↓
Configure AWS Credentials
      ↓
terraform init
      ↓
terraform fmt
      ↓
terraform validate
      ↓
terraform plan
      ↓
Review
      ↓
terraform apply
      ↓
EC2 Created
      ↓
terraform output
      ↓
Use EC2
      ↓
terraform destroy
```

---

# 🛠️ Important Terraform Commands

Version:

```bash
terraform --version
```

Help:

```bash
terraform -help
```

Initialize:

```bash
terraform init
```

Format:

```bash
terraform fmt
```

Format check:

```bash
terraform fmt -check
```

Validate:

```bash
terraform validate
```

Plan:

```bash
terraform plan
```

Apply:

```bash
terraform apply
```

Apply without confirmation:

```bash
terraform apply -auto-approve
```

Show state:

```bash
terraform show
```

List resources:

```bash
terraform state list
```

Show resource:

```bash
terraform state show aws_instance.web
```

Outputs:

```bash
terraform output
```

Destroy:

```bash
terraform destroy
```

---

# 🚨 Common Error — Terraform Not Found

Windows:

```powershell
Get-Command terraform
```

Linux/macOS:

```bash
which terraform
```

Check:

```bash
terraform --version
```

If not found:

```text
Check installation
Check PATH
Open a new terminal
```

---

# 🚨 Common Error — AWS Credentials

Run:

```bash
aws sts get-caller-identity
```

If it fails:

```text
Check credentials
Check profile
Check IAM permissions
Check environment variables
Check region
```

---

# 🚨 Common Error — UnauthorizedOperation

Possible reasons:

```text
IAM policy
SCP
Permission boundary
Resource policy
Wrong AWS account
Wrong region
```

Check the identity:

```bash
aws sts get-caller-identity
```

---

# 🚨 Common Error — Invalid AMI

AMI IDs are region-specific.

Example:

```text
AMI in ap-south-1
```

does not automatically work in:

```text
us-east-1
```

Find an AMI valid for your selected region.

---

# 🚨 Common Error — Key Pair Not Found

If Terraform has:

```hcl
key_name = "my-key"
```

AWS must have an existing EC2 key pair with that name in the same region/account.

---

# 🚨 Common Error — Provider Initialization

Run:

```bash
terraform init
```

If provider upgrades are intentionally needed:

```bash
terraform init -upgrade
```

Use `-upgrade` deliberately because provider selections can change within your declared constraints.

---

# 🔐 Terraform Security

Never hardcode:

```hcl
provider "aws" {
  access_key = "AKIA..."
  secret_key = "SECRET..."
}
```

Never commit:

```text
AWS Access Key
AWS Secret Key
Private SSH Key
Passwords
API Tokens
Terraform State
Secret tfvars
```

---

# 👤 Better AWS Authentication

For EC2:

```text
EC2
 ↓
IAM Role / Instance Profile
 ↓
Temporary Credentials
 ↓
AWS API
```

For CI/CD:

```text
CI/CD
 ↓
OIDC
 ↓
AWS IAM Role
 ↓
Temporary Credentials
 ↓
AWS API
```

For developers:

```text
AWS IAM Identity Center
        or
AWS CLI Profile
        or
Short-lived credentials
```

---

# 🔒 Least Privilege

Do not automatically give Terraform:

```text
AdministratorAccess
```

unless it is genuinely required.

Prefer:

```text
Only required permissions
```

Example concept:

```text
Terraform
   ↓
Only required AWS APIs
   ↓
Create required infrastructure
```

---

# 🐙 Terraform + Git

Terraform code should normally be stored in Git.

Example:

```text
Developer
    ↓
Terraform Code
    ↓
Git
    ↓
Pull Request
    ↓
Code Review
    ↓
CI
    ↓
terraform plan
    ↓
Approval
    ↓
terraform apply
```

---

# 🚀 Terraform + CI/CD

A common pipeline:

```text
Git Push
   ↓
CI Pipeline
   ↓
terraform fmt -check
   ↓
terraform init
   ↓
terraform validate
   ↓
Security Scan
   ↓
terraform plan
   ↓
Pull Request Review
   ↓
Approval
   ↓
terraform apply
```

Possible security/policy tools:

```text
Checkov
Trivy
TFLint
OPA / Conftest
```

---

# 🤝 Terraform + Ansible

Terraform:

```text
Provision Infrastructure
```

Ansible:

```text
Configure Servers
```

Example:

```text
Git
 ↓
Terraform
 ↓
VPC
 ↓
Subnet
 ↓
Security Group
 ↓
EC2
 ↓
Output IP
 ↓
Ansible
 ↓
Install Docker
 ↓
Install Nginx
 ↓
Deploy Application
```

---

# 🏭 Production Terraform Workflow

```text
Developer
    ↓
Terraform Code
    ↓
Git Branch
    ↓
Pull Request
    ↓
CI
    ├── fmt
    ├── validate
    ├── security scan
    ├── policy check
    └── plan
    ↓
Code Review
    ↓
Approval
    ↓
Controlled Apply
    ↓
Cloud Infrastructure
```

Production practices:

```text
Remote State
State Encryption
Access Control
Provider Version Constraints
.terraform.lock.hcl
Modules
Environment Separation
Pull Request Review
Security Scanning
Least Privilege
Audit Logging
Controlled Production Apply
```

---

# 🗄️ Remote State

Local state:

```text
terraform.tfstate
```

Production commonly uses a remote backend.

Concept:

```text
Developer / CI
      ↓
Terraform
      ↓
Remote State Backend
      ↓
Shared State
```

Benefits:

```text
Team collaboration
Centralized state
Access control
State protection
```

The exact backend depends on your architecture and Terraform version.

---

# 📦 Terraform Modules

A module is a reusable Terraform configuration.

Example:

```text
modules/
   ├── network/
   └── ec2/
```

Network module:

```text
VPC
Subnet
Route Table
Gateway
```

EC2 module:

```text
EC2
Security Group
```

Then:

```text
Root Module
    ↓
Network Module
    ↓
EC2 Module
```

Modules help reduce repeated code.

---

# 🌍 Environment Structure

Example:

```text
environments/
   ├── dev/
   ├── staging/
   └── prod/
```

Concept:

```text
Same infrastructure design
        ↓
Different environment values
        ↓
Dev / Staging / Production
```

Be careful with state isolation so one environment cannot accidentally modify another.

---

# 🧠 Terraform Interview Revision

| Component | Purpose | Easy Analogy |
|---|---|---|
| Terraform | IaC tool | Infrastructure Engineer |
| HCL | Configuration language | Blueprint Language |
| Provider | Connects to platform | Translator |
| Resource | Managed infrastructure | Building |
| Variable | Input | Form Input |
| Output | Exposed result | Report |
| Local | Reusable expression | Shortcut |
| Data Source | Reads existing information | Information Desk |
| State | Tracks managed infrastructure | Memory |
| Module | Reusable configuration | Template |
| Plan | Shows proposed changes | Preview |
| Apply | Makes changes | Execute |
| Destroy | Removes managed resources | Demolish |

---

# 🎯 Key Terraform Points

- Terraform is an Infrastructure as Code tool.
- Terraform is primarily declarative.
- Terraform commonly uses HCL.
- Providers connect Terraform to external platforms.
- Resources represent infrastructure Terraform manages.
- Variables provide inputs.
- Outputs expose useful values.
- Locals simplify repeated expressions.
- Data sources read existing information.
- State tracks Terraform-managed infrastructure.
- `terraform init` initializes the working directory.
- `terraform fmt` formats Terraform code.
- `terraform validate` validates configuration.
- `terraform plan` previews changes.
- `terraform apply` applies changes.
- `terraform destroy` removes managed infrastructure.
- Terraform can manage AWS, Azure, GCP, Alibaba Cloud and many other platforms.
- Terraform and Ansible can complement each other.
- CloudFormation is AWS-native IaC.
- OpenTofu is an open-source Terraform fork.
- Terraform's newer releases use the Business Source License 1.1.
- Older Terraform releases remain under their original licenses.
- Never commit credentials or sensitive Terraform state.
- Review `terraform plan` before production changes.
- Use remote state and controlled access in team environments.

---

# 🧠 Complete Terraform Mental Model

```text
                 INFRASTRUCTURE AS CODE
                          │
                          ▼
                      TERRAFORM
                          │
              +-----------+-----------+
              │                       │
              ▼                       ▼
             HCL                 Terraform CLI
              │                       │
              +-----------+-----------+
                          │
                          ▼
                       Provider
                          │
              +-----------+-----------+
              │           │           │
              ▼           ▼           ▼
             AWS         Azure       GCP
              │
              ▼
          Cloud APIs
              │
       +------+------+------+
       │      │      │      │
       ▼      ▼      ▼      ▼
      VPC    EC2     S3     RDS
```

---

# 🔄 Terraform DevOps Mental Model

```text
                         GIT
                          │
                          ▼
                     CI/CD Pipeline
                          │
             +------------+------------+
             │                         │
             ▼                         ▼
        Terraform                 Application
             │                         │
             ▼                         ▼
      Cloud Infrastructure          Docker
             │                         │
             ▼                         ▼
         Kubernetes              Application
             │
             ▼
          Servers
```

---

# 🧭 Terraform Learning Roadmap

Follow this order:

```text
1. Cloud Fundamentals
        ↓
2. AWS EC2
        ↓
3. AWS VPC
        ↓
4. AWS CLI
        ↓
5. Terraform Installation
        ↓
6. HCL
        ↓
7. Provider
        ↓
8. Resource
        ↓
9. Variables
        ↓
10. Outputs
        ↓
11. Locals
        ↓
12. Data Sources
        ↓
13. State
        ↓
14. Modules
        ↓
15. Remote State
        ↓
16. Security
        ↓
17. Git
        ↓
18. CI/CD
        ↓
19. Terraform + Ansible
        ↓
20. Complete AWS Project
```

---

# 🚀 Beginner Project

Build:

```text
Terraform
   ↓
AWS Provider
   ↓
EC2
   ↓
Output Public IP
   ↓
SSH
```

---

# 🚀 Intermediate Project

Build:

```text
Terraform
   ↓
VPC
   ├── Public Subnet
   ├── Private Subnet
   ├── Internet Gateway
   ├── NAT Gateway
   ├── Route Tables
   ├── Security Groups
   ├── EC2
   └── Load Balancer
```

---

# 🚀 Advanced Project

Build:

```text
Terraform
   ├── VPC
   ├── EKS
   ├── IAM
   ├── RDS
   ├── S3
   ├── Load Balancer
   ├── CloudWatch
   └── Security
             ↓
        Kubernetes
             ↓
           CI/CD
             ↓
        Application
```

---

# ✅ Terraform Learning Checklist

- [ ] Understand Terraform
- [ ] Understand IaC
- [ ] Understand declarative configuration
- [ ] Understand HCL
- [ ] Understand providers
- [ ] Understand resources
- [ ] Understand variables
- [ ] Understand outputs
- [ ] Understand locals
- [ ] Understand data sources
- [ ] Understand state
- [ ] Understand dependency graph
- [ ] Learn `terraform init`
- [ ] Learn `terraform fmt`
- [ ] Learn `terraform validate`
- [ ] Learn `terraform plan`
- [ ] Learn `terraform apply`
- [ ] Learn `terraform destroy`
- [ ] Install Terraform on Windows
- [ ] Install Terraform on macOS
- [ ] Install Terraform on Linux
- [ ] Install Terraform on EC2
- [ ] Install AWS CLI
- [ ] Configure AWS authentication
- [ ] Create EC2 using Terraform
- [ ] Learn Terraform vs Ansible
- [ ] Learn Terraform vs CloudFormation
- [ ] Learn Terraform vs OpenTofu
- [ ] Learn Terraform vs Pulumi
- [ ] Learn Terraform vs Crossplane
- [ ] Learn modules
- [ ] Learn remote state
- [ ] Learn Git workflow
- [ ] Learn CI/CD
- [ ] Learn Terraform security scanning
- [ ] Build an end-to-end AWS project

---

# ⚠️ AWS Cost Warning

Terraform can create real AWS resources.

Before:

```bash
terraform apply
```

always run:

```bash
terraform plan
```

After a temporary learning project:

```bash
terraform destroy
```

Also check the AWS billing console.

Do not leave unnecessary resources running.

---

# 🔐 Security Warning

Never publish:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
Private SSH Keys
Passwords
Tokens
Terraform State
Secret Variables
```

Use:

```text
IAM Roles
Short-lived Credentials
AWS Identity Center
OIDC
Environment Variables
Secure CI/CD Secrets
```

according to the environment.

---

# 🌐 Official Documentation

Terraform:

```text
https://developer.hashicorp.com/terraform
```

Terraform Installation:

```text
https://developer.hashicorp.com/terraform/install
```

Terraform Language:

```text
https://developer.hashicorp.com/terraform/language
```

Terraform Registry:

```text
https://registry.terraform.io/
```

AWS Provider:

```text
https://registry.terraform.io/providers/hashicorp/aws/latest
```

OpenTofu:

```text
https://opentofu.org/
```

OpenTofu Documentation:

```text
https://opentofu.org/docs/
```

AWS CLI:

```text
https://docs.aws.amazon.com/cli/
```

AWS CLI Configuration:

```text
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
```

---

# 🎯 Final One-Line Revision

```text
Terraform = Declarative Infrastructure as Code
          ↓
          HCL
          ↓
       Provider
          ↓
       Resources
          ↓
     terraform plan
          ↓
    terraform apply
          ↓
    Cloud Infrastructure
```

---

# ⭐ Most Important Interview Answer

> **Terraform is a declarative Infrastructure as Code tool that allows engineers to define desired infrastructure in configuration files. Terraform uses providers to communicate with platforms such as AWS, Azure and GCP, maintains state to track managed infrastructure, and provides a workflow such as `init → plan → apply` to safely provision and manage infrastructure.**

---

# 📌 GitHub Repository Suggested Structure

```text
terraform-learning/
│
├── README.md
│
├── 01-terraform-basics/
│   ├── main.tf
│   ├── providers.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── 02-aws-ec2/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
│
├── 03-aws-vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
│
├── 04-modules/
│   ├── modules/
│   │   ├── vpc/
│   │   └── ec2/
│   └── README.md
│
├── 05-environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
│
└── .gitignore
```

Recommended `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json
crash.log
crash.*.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```

Commit:

```text
README.md
*.tf
.terraform.lock.hcl
terraform.tfvars.example
```

Do not commit:

```text
terraform.tfstate
terraform.tfstate.backup
real secret tfvars
AWS credentials
private keys
```

---

# 🏁 End-to-End Summary

```text
Developer
    │
    ▼
Write Terraform HCL
    │
    ▼
terraform fmt
    │
    ▼
terraform validate
    │
    ▼
terraform init
    │
    ▼
Provider Download
    │
    ▼
terraform plan
    │
    ▼
Review Changes
    │
    ▼
terraform apply
    │
    ▼
Provider API
    │
    ▼
AWS / Azure / GCP
    │
    ▼
Infrastructure
    │
    ▼
Terraform State
    │
    ▼
Git + CI/CD
    │
    ▼
Production Infrastructure
```

## The Core Concept

```text
Terraform tells the cloud:

"What should my infrastructure look like?"

Terraform then calculates:

"What changes are required?"

The provider performs:

"The required API operations."

The state records:

"What Terraform knows about the infrastructure."
```

**This is the core mental model to remember while learning Terraform.**
