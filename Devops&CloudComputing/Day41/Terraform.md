# 🌍 Terraform (Infrastructure as Code) — Notes

## 1. Introduction to IaC (Infrastructure as Code)

**Definition:**  
Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure using code instead of manual processes.

**Why IaC?**
- Consistency (eliminates manual errors)
- Version control (Git integration)
- Automation (CI/CD pipelines)
- Reusability
- Faster provisioning

**Types of IaC:**
- **Declarative:** Define *what* you want (Terraform)
- **Imperative:** Define *how* to do it (Shell, Python scripts)

---

## 2. Introduction to Terraform

Terraform is an open-source IaC tool by HashiCorp used to provision infrastructure across multiple cloud providers.

**Key Features:**
- Cloud-agnostic (AWS, Azure, GCP, etc.)
- Declarative configuration
- Execution plan before applying changes
- State tracking

**Basic Workflow:**
```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

---

## 3. HCL (HashiCorp Configuration Language)

Terraform uses HCL to define infrastructure.

**Basic Syntax:**
```hcl
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

**Components:**
- Blocks (`resource`, `provider`, `variable`)
- Arguments (`key = value`)
- Expressions (`var.name`, `count.index`)

**Data Types:**
- string, number, bool
- list, map, object

---

## 4. Providers

Providers are plugins that allow Terraform to interact with cloud platforms.

```hcl
provider "aws" {
  region = "us-east-1"
}
```

**Key Points:**
- Installed via `terraform init`
- Support authentication (access keys, profiles)
- Multiple providers can be used

---

## 5. Resources

Resources are the actual infrastructure components.

```hcl
resource "aws_s3_bucket" "bucket" {
  bucket = "my-bucket"
}
```

**Syntax:**
```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  ...
}
```

**Meta-arguments:**
- `count`
- `for_each`
- `depends_on`
- `lifecycle`

---

## 6. Variables

Used to make configurations reusable.

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

**Usage:**
```hcl
instance_type = var.instance_type
```

**Types:**
- string, number, bool
- list, map

**Ways to Assign Values:**
- `.tfvars` file
- CLI (`-var`)
- Environment variables

---

## 7. Outputs

Used to display values after execution.

```hcl
output "instance_ip" {
  value = aws_instance.example.public_ip
}
```

**Use Cases:**
- Debugging
- Sharing values between modules
- CI/CD pipelines

---

## 8. State Management

Terraform stores infrastructure state in `terraform.tfstate`.

**Why Important:**
- Maps real infrastructure to configuration
- Tracks changes
- Enables incremental updates

**Commands:**
```bash
terraform state list
terraform state show <resource>
```

**Important Notes:**
- Do not edit manually
- May contain sensitive data

---

## 9. Remote State (S3)

Local state is not suitable for teams. Use remote storage.

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "dev/terraform.tfstate"
    region = "us-east-1"
  }
}
```

**State Locking (Recommended):**
```hcl
dynamodb_table = "terraform-lock"
```

**Benefits:**
- Shared access
- Prevents conflicts
- Secure storage

---

## 10. Modules

Modules are reusable Terraform configurations.

```hcl
module "vpc" {
  source = "./modules/vpc"
}
```

**Types:**
- Local modules
- Remote modules (Git, Terraform Registry)

**Benefits:**
- Code reusability
- Better organization
- Scalability

**Structure:**
```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
```

---

## 11. Workspaces

Workspaces allow managing multiple environments.

```bash
terraform workspace new dev
terraform workspace select dev
```

**Example:**
```hcl
bucket = "my-app-${terraform.workspace}"
```

**Use Case:**
- Separate dev, staging, and prod environments

---

## 12. Best Practices

- Use remote state with locking
- Keep configurations modular
- Avoid hardcoding values
- Use `.tfvars` for environment configs
- Store secrets securely (env variables, Vault)
- Use version control (Git)
- Run:
  ```bash
  terraform fmt
  terraform validate
  ```

---

## 13. Terraform Lifecycle Summary

1. Write configuration files (`.tf`)
2. Initialize → `terraform init`
3. Plan → `terraform plan`
4. Apply → `terraform apply`
5. Destroy → `terraform destroy`

---