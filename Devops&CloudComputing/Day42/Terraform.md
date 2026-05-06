## 🌍 Terraform (Infrastructure as Code) 
------------------------------------------------------------------
## 1. Introduction to IaC (Infrastructure as Code)
Infrastructure as Code (IaC) is the practice of managing and provisioning IT infrastructure (networks, VMs, load balancers, etc.) using machine-readable definition files, rather than manual hardware configuration or web-based console tools.
------------------------------------------------------------------
Q.Why IaC?

- Consistency: - Eliminates "Environmental Drift" by ensuring the Dev, Test, and Prod environments are identical.
- Idempotency: - Running the same code multiple times always yields the same result without creating duplicate resources.
- Speed & Automation: - Integrated into CI/CD pipelines for rapid, hands-off provisioning.
- Version Control: - Infrastructure is stored in Git, allowing for change tracking, peer reviews, and easy "rollbacks" to previous states.
- Cost Efficiency: - Environments can be spun up only when needed and destroyed immediately after, reducing cloud waste.

** Core Paradigms:-

- Declarative (The "What"): You define the desired end-state (e.g., "I want 3 EC2 instances").
  The tool figures out how to make it happen.
  Ex:- Terraform, CloudFormation, Kubernetes manifests.
- Imperative (The "How"): You define the specific commands and steps to reach the state (e.g., "Step 1: Create VPC, Step 2: Create Subnet").
  Ex:- AWS CLI, Bash scripts, Python (Boto3).
----------------------------------------------------------------------
**Types of IaC:**

- **Declarative:** Define _what_ you want (Terraform)
- **Imperative:** Define _how_ to do it (Shell, Python scripts)
---------------------------------------------------------------------
## 2. Introduction to Terraform

Terraform is an open-source Infrastructure as Code (IaC) tool created by HashiCorp.
It allows you to define both **cloud** and **on-premise** resources in human-readable configuration files that you can version, reuse, and share.

**Key Features:**

- Cloud-Agnostic: A single tool to manage multiple providers (AWS, Azure, GCP, DigitalOcean) and even SaaS providers (Cloudflare, GitHub).

- Declarative Language: Uses HCL (HashiCorp Configuration Language). You describe the "Desired State," and Terraform handles the underlying logic to achieve it.

- Execution Plan: Generates a preview (plan) of exactly what will be created, modified, or destroyed before any real-world changes are made.

- Resource Graph: Builds a dependency graph to manage resources in the correct order (e.g., ensuring a VPC is created before the Subnet).

Core Components:-

- Providers: Plugins that allow Terraform to interact with cloud providers and APIs.
- State File (terraform.tfstate): A JSON file that maps your HCL code to real-world resources.
 - It is the "source of truth" for your infrastructure.
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
human-readable, **declarative language** designed specifically for infrastructure automation and is the primary language used by Terraform.
It strikes a balance between being easy for humans to read and write while staying structured enough for machines to process efficiently.
Instead of writing step-by-step instructions, students use HCL to describe the "desired state" of their cloud resources, and the tool handles the creation and management of those assets.

**Basic Syntax:**

# Resource Type: "aws_instance" (What AWS service are we using?)
# Local Name: "example" (How do we refer to this in our code?)
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # The Operating System ID
  instance_type = "t2.micro"             # The hardware size (Free Tier)

  tags = {
    Name = "MyFirstServer"
  }
}
```
Resources are the actual infrastructure components.
----------------------------------------------------------------------------
**Components:**
Blocks: These are containers for other content and represent the "objects" you are managing, such as a resource (the infrastructure to build), a provider (the cloud platform like AWS), or a variable (input values).  
Arguments: These exist inside blocks to assign values to specific names using the key = value syntax (e.g., instance_type = "t2.micro").  
Expressions: These are used to represent values dynamically, such as referencing a variable (var.instance_name) or using built-in logic like count.index to create multiple resources at once.

- Blocks (`resource`, `provider`, `variable`)
- Arguments (`key = value`)
- Expressions (`var.name`, `count.index`)

**Data Types:**
Primitive Types:
string: A sequence of characters wrapped in quotes (e.g., "ami-12345").
number: Numeric values used for counts or sizes (e.g., 2 or 0.5).
bool: A simple true or false value for toggling features.

Complex Types:
list: A sequential collection of values (e.g., ["subnet-1", "subnet-2"]).
map: A collection of key-value pairs (e.g., { environment = "dev", project = "web" }).
object: A structured collection of different types of values, used for more complex configurations.
----------------------------------------------------------------------------
## 4. Providers

Providers are plugins that allow Terraform to interact with cloud platforms.

```hcl
provider "aws" {
  region = "us-east-1"
}
```
----------------------------------------------------------------------------
**Key Points:**

- Installed via `terraform init`
- Support authentication (access keys, profiles)
- Multiple providers can be used
-----------------------------------------------------------------------------
**Syntax:**

```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  ...
}
```
-----------------------------------------------------------------------------
**Meta-arguments:**
Meta-arguments are special arguments that can be used within any resource block to change how Terraform handles that resource. They are not specific to any provider and can be used with any resource type.

count: Allows you to create a specific number of identical resources without writing separate code blocks for each one.  
for_each: Similar to count, but allows you to create multiple resources based on a map or a set of strings, which is better for unique naming.  
depends_on: Explicitly defines a hidden dependency, telling Terraform that one resource must be created only after another specific resource is finished.  
lifecycle: A nested block used to control the behavior of resources, such as preventing a resource from being accidentally deleted or ensuring a new one is created before the old one is destroyed.
- `count`
- `for_each`
- `depends_on`
- `lifecycle`

----------------------------------------------------------------------------
## 5. Variables

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
----------------------------------------------------------------------------
## 6. Outputs

Used to display values after execution.

```hcl
output "instance_ip" {
  value = aws_instance.example.public_ip
}
```
-----------------------------------------------------------------------------
**Use Cases:**

- Debugging
- Sharing values between modules
- CI/CD pipelines

----------------------------------------------------------------------------
## 7 . State Management

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
----------------------------------------------------------------------------
## 8. Remote State (S3)

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
----------------------------------------------------------------------------
## 9 . Modules

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
------------------------------------------------------------------------------
## 10. Workspaces

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

------------------------------------------------------------------------------
## 11. Best Practices

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
------------------------------------------------------------------------------
## 12. Terraform Lifecycle Summary

1. Write configuration files (`.tf`)
2. Initialize → `terraform init`
3. Plan → `terraform plan`
4. Apply → `terraform apply`
5. Destroy → `terraform destroy`
------------------------------------------------------------------------------
