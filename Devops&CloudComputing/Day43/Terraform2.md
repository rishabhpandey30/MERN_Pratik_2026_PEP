🚀 Terraform
--------------------------------------------------------
Terraform is an Infrastructure as Code (IaC) tool used to create, manage, and automate cloud infrastructure using code. It allows you to define your infrastructure in a high-level configuration language, which can be versioned and reused.

Ex:-

EC2 instances
VPC
S3 buckets
Load balancers
Databases
--------------------------------------------------------
Q.Why do we use Terraform?

Without Terraform:-

- Manual cloud setup: We were setting up cloud resources manually through the cloud provider's console, which was time-consuming and error-prone.
- Repetitive work: Every time we needed to create similar infrastructure, we had to repeat the same steps, leading to inefficiency.
- Human errors: Manual setup often resulted in mistakes, such as misconfigurations or forgetting to set up certain resources.
- Hard to track changes: Without version control, it was difficult to track changes made to the infrastructure, leading to confusion and potential issues when troubleshooting.

With Terraform:-
- Infrastructure in code: We can define our infrastructure using code, making it easier to manage and understand.
- Reusable: We can reuse our Terraform configurations to create similar infrastructure across different environments (e.g., development, staging, production) without having to repeat the setup process.
- Version controlled: We can use version control systems (like Git) to track changes to our infrastructure code, making it easier to collaborate and troubleshoot issues. 
- Fast deployment: Terraform allows us to quickly deploy and manage infrastructure, reducing the time it takes to set up and maintain our cloud resources.
- Automation friendly: Terraform can be integrated into CI/CD pipelines, enabling automated infrastructure provisioning and management, which further enhances efficiency and reduces the chances of human error.

--------------------------------------------------------
## Real Life Analogy:
Manual Setup:
Like assembling furniture manually every time.

With Terraform:
Like having an instruction blueprint that builds everything automatically.
---------------------------------------------------------
Q. What Terraform is Used For ?
Terraform is used for various purposes in cloud infrastructure management. Here are some common use cases:

  Use Case	                                           Example
 Cloud Provisioning	                                   Create EC2, VPC, S3
 Multi-cloud Setup	                                   AWS + Azure + GCP
 Automation	                                           CI/CD deployments
 Infrastructure Replication	                           Same infra for Dev/Test/Prod
 Scaling	                                               Create multiple servers quickly

 ---------------------------------------------------------
 ## Terraform Architecture:
Terraform has a modular architecture that consists of several components:
- Providers: These are plugins that allow Terraform to interact with different cloud platforms and services (e.g., AWS, Azure, GCP).
- Resources: These are the components of your infrastructure that you want to create or manage (e.g., EC2 instances, S3 buckets).
- State: Terraform keeps track of the current state of your infrastructure in a state file, which allows it to determine what changes need to  be made when you apply your configuration.
- Modules: These are reusable configurations that can be shared and used across different projects or environments.
- CLI: The command-line interface that allows you to interact with Terraform and manage your infrastructure.
-----------------------------------------------------------
## Terraform Workflow:
1. Write: You write your infrastructure configuration in a .tf file using the HashiCorp Configuration Language (HCL).
  command: `terraform init` to initialize the working directory and download necessary providers.
2. Plan: You run `terraform plan` to see what changes Terraform will make to your infrastructure based on your configuration.
3. Apply: You run `terraform apply` to apply the changes and create or update your infrastructure.
4. Destroy: You can run `terraform destroy` to tear down your infrastructure when it's no longer needed.
-----------------------------------------------------------
## Terraform Lifecycle:
This is the lifecycle of a Terraform resource:

Write Code
   ↓
terraform init
   ↓
terraform plan
   ↓
terraform apply
   ↓
Infrastructure Created
   
-----------------------------------------------------------
## Basic Terraform File Structure:
- main.tf: This is the main configuration file where you define your resources and infrastructure.

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "demo" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
}
```
- variables.tf: This file is used to define input variables that can be used in your configuration.

```
variable "region" {
  description = "The AWS region to deploy resources"
  default     = "us-east-1"
}
```
- outputs.tf: This file is used to define output values that can be displayed after applying your configuration.

```
output "instance_id" {
  description = "The ID of the created EC2 instance"
  value       = aws_instance.demo.id
}
```
- terraform.tfvars: This file is used to provide values for the input variables defined in variables.tf.

```
region = "us-west-2"
```
-------------------------------------------------------------
# state file:

This file (terraform.tfstate) is automatically generated by Terraform to keep track of the current state of your infrastructure. It should not be manually edited and should be stored securely, especially if it contains sensitive information.

It Stores:

Resource IDs
Current infrastructure info
Mapping between code and real infra

Without state:
Terraform won’t know what already exists.
-------------------------------------------------------------