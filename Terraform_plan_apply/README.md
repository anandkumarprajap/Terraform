# Terraform Workflow: Step-by-Step Guide

## 1. Understand `terraform init`

`terraform init` initializes the Terraform working directory. It downloads required providers, installs modules, and initializes the backend.

**Important rule:** Do not run `terraform init` every time you modify a Terraform configuration block. Run it when initialization-related changes require it.

## 2. Step-by-Step Terraform Workflow

### Step 1: Create Terraform Configuration

Create your Terraform configuration file.

```bash
mkdir terraform-demo
cd terraform-demo
touch main.tf
```

Example `main.tf`:

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "demo" {
  bucket = "my-unique-demo-bucket-12345"
}
```

Replace the bucket name with a globally unique name before applying.

### Step 2: Initialize Terraform

Run:

```bash
terraform init
```

**Why?**
- Downloads the required provider.
- Initializes the working directory.
- Prepares Terraform to manage infrastructure.

Run this step when starting a new working directory or when initialization-related configuration changes require it.

### Step 3: Format the Configuration

```bash
terraform fmt
```

**Why?** Formats Terraform files into a consistent style.

### Step 4: Validate the Configuration

```bash
terraform validate
```

**Why?** Checks whether the configuration is syntactically valid and internally consistent.

### Step 5: Preview Infrastructure Changes

```bash
terraform plan
```

**Why?** Shows what Terraform intends to create, update, or destroy.

Review the plan before proceeding.

### Step 6: Apply the Configuration

```bash
terraform apply
```

Review the proposed changes and type `yes` when prompted if you want to proceed.

**Result:** Terraform applies the planned infrastructure changes.

## 3. When Should You Run `terraform init` Again?

| Configuration change | Run `terraform init`? |
|---|---|
| Update a `resource` block | Usually no |
| Update a `data` block | Usually no |
| Update an `output` block | Usually no |
| Update a `variable` block | Usually no |
| Add or change provider requirements | Yes, when reinitialization is required |
| Add a module or change its source | Yes, when reinitialization is required |
| Change backend configuration | Yes, to reinitialize or migrate backend settings as appropriate |

If Terraform reports that initialization is required, run `terraform init` before continuing.

## 4. Workflow After Updating a Configuration Block

For ordinary changes to a resource, data source, output, or variable, use this workflow:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

You generally do not need to run `terraform init` for these edits alone.

## 5. Complete Terraform Workflow

```text
Create or update configuration
             |
             v
      terraform init
   (when initialization
      is required)
             |
             v
       terraform fmt
             |
             v
      terraform validate
             |
             v
       terraform plan
             |
             v
Review the proposed changes
             |
             v
       terraform apply
             |
             v
Infrastructure updated
```

For later configuration updates, repeat the formatting, validation, planning, and application steps. Reinitialize only when required.

## 6. Quick Revision

- `terraform init` → Initialize providers, modules, and backend.
- `terraform fmt` → Format configuration files.
- `terraform validate` → Check configuration validity.
- `terraform plan` → Preview infrastructure changes.
- `terraform apply` → Apply infrastructure changes.

**Golden rule:** Resource, data, output, and variable block edits alone generally do not require `terraform init`. Provider, module, and backend changes may require it.
```txt
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
```
# Terraform Init – When to Run `terraform init`

## 1. Definition

`terraform init` initializes a Terraform working directory. It downloads required providers, installs modules, and initializes the configured backend.

## 2. When Do We Need to Run `terraform init`?

Run `terraform init` when initialization-related configuration changes require it.

Examples:
- Adding or changing provider requirements.
- Adding a new module or changing a module source.
- Changing the backend configuration.
- Reinitializing the working directory when Terraform requires it.

## 3. When Is `terraform init` Not Required?

You generally do **not** need to run `terraform init` just because you modify the following blocks:

| Terraform block | Is `terraform init` required? |
|---|---|
| `resource` block | No |
| `data` block | No |
| `output` block | No |
| `variable` block | No |

These changes generally require `terraform plan` to review the changes and `terraform apply` to apply them, as appropriate.

## 4. Example

### Resource Block

```hcl
resource "aws_s3_bucket" "demo" {
  bucket = "my-demo-bucket-12345"
}
```

If you update this resource block, you generally do not need to run `terraform init`.

Run:

```bash
terraform plan
terraform apply
```

Terraform will show the proposed changes before applying them.

## 5. Important Terraform Commands

| Command | Purpose |
|---|---|
| `terraform init` | Initialize the working directory |
| `terraform fmt` | Format Terraform configuration files |
| `terraform validate` | Validate configuration syntax and structure |
| `terraform plan` | Preview proposed infrastructure changes |
| `terraform apply` | Apply infrastructure changes |
| `terraform destroy` | Destroy managed infrastructure |

## 6. Quick Revision

**Remember this rule:**

- Provider, module, or backend initialization changes → Run `terraform init` when required.
- Resource, data, output, or variable block changes → Usually no need to run `terraform init`.
- Review infrastructure changes → `terraform plan`.
- Apply infrastructure changes → `terraform apply`.

**Note:** These rules are general guidance. If Terraform reports that initialization is required, run `terraform init` before continuing.
