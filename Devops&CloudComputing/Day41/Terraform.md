## 🌍 Terraform (Infrastructure as Code) 
------------------------------------------------------------------
## 1. Introduction to IaC (Infrastructure as Code)
Infrastructure as Code (IaC) is the practice of managing and provisioning IT infrastructure (networks, VMs, load balancers, etc.) using machine-readable definition files, rather than manual hardware configuration or web-based console tools.
------------------------------------------------------------------
Q.Why IaC?

- Consistency: Eliminates "Environmental Drift" by ensuring the Dev, Test, and Prod environments are identical.
- Idempotency: Running the same code multiple times always yields the same result without creating duplicate resources.
- Speed & Automation: Integrated into CI/CD pipelines for rapid, hands-off provisioning.
- Version Control: Infrastructure is stored in Git, allowing for change tracking, peer reviews, and easy "rollbacks" to previous states.
- Cost Efficiency: Environments can be spun up only when needed and destroyed immediately after, reducing cloud waste.

** Core Paradigms:-

- Declarative (The "What"): You define the desired end-state (e.g., "I want 3 EC2 instances").
  The tool figures out how to make it happen.
  Ex:- Terraform, CloudFormation, Kubernetes manifests.
- Imperative (The "How"): You define the specific commands and steps to reach the state (e.g., "Step 1: Create VPC, Step 2: Create Subnet").
  Exa:- AWS CLI, Bash scripts, Python (Boto3).
----------------------------------------------------------------------
**Types of IaC:**

- **Declarative:** Define _what_ you want (Terraform)
- **Imperative:** Define _how_ to do it (Shell, Python scripts)
---------------------------------------------------------------------
## 2. Introduction to Terraform

Terraform is an open-source Infrastructure as Code (IaC) tool created by HashiCorp.
It allows you to define both cloud and on-premise resources in human-readable configuration files that you can version, reuse, and share.

**Key Features:**

- Cloud-Agnostic: A single tool to manage multiple providers (AWS, Azure, GCP, DigitalOcean) and even SaaS providers (Cloudflare, GitHub).

- Declarative Language: Uses HCL (HashiCorp Configuration Language). You describe the "Desired State," and Terraform handles the underlying logic to achieve it.

- Execution Plan: Generates a preview (plan) of exactly what will be created, modified, or destroyed before any real-world changes are made.

- Resource Graph: Builds a dependency graph to manage resources in the correct order (e.g., ensuring a VPC is created before the Subnet).

Core Components:-

- Providers: Plugins that allow Terraform to interact with cloud providers and APIs.
- State File (terraform.tfstate): A JSON file that maps your HCL code to real-world resources.
  It is the "source of truth" for your infrastructure.
- Modules: Containers for multiple resources that are used together. They allow you to package and reuse infrastructure code like a library.
---------------------------------------------------------------------------
**Basic Workflow:**
| Command | Action | Description |
| :--- |  :--- | :--- |
| `terraform init` | **Initialize** | Prepares the working directory, downloads the necessary Provider plugins, and initializes the backend. |
| `terraform plan` | **Preview** | Compares your local HCL code against the current cloud state and creates an execution plan. |
| `terraform apply` | **Execute** | Applies the changes defined in your code to reach the desired state in the cloud. |
| `terraform destroy` | **Clean Up** | Identifies all resources managed in the state file and deletes them from the cloud provider. |                                   

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```
----------------------------------------------------------------------------
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
