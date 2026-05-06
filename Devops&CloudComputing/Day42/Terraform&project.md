## Terraform & HCL Master Guide: From Fundamentals to Live Deployment:
--------------------------------------------------------------------------
## The Terraform Workflow (Lifecycle):

- terraform init: Downloads the necessary Cloud Providers (AWS, etc.).  
- terraform plan: Previews exactly what will be created or changed.  
- terraform apply: Executes the plan and builds the live infrastructure.  
- terraform destroy: Wipes out all managed resources to stop billing.
--------------------------------------------------------------------------
## Project Deployment with (Docker + EC2 + Terraform):

## Step 1:Initialize AWS Credentials

- Before running any automation tools, your local environment must have permission to manage AWS resources.
- Open your terminal and run: aws configure
- verify the access keys by running: aws sts get-caller-identity (if you have already configured the keys in AWS CLI)
- Enter your AWS Access Key ID and AWS Secret Access Key.
- Set the Default region name to us-east-1 (or your preferred region).
--------------------------------------------------------------------------
## Step 2: Containerize the Project:
(a).Create a Dockerfile in your project's root folder with this content:

```Dockerfile

FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```
(b).Build your image:
docker build -t username/blog-app .

(c)Push to Docker Hub (ensure the repository is set to Public):
docker push username/blog-app
--------------------------------------------------------------------------
## Step 3: Write Terraform Configuration:

(a). Create a file named main.tf in a separate folder. This code acts as the blueprint for your cloud hardware.

```hcl 

provider "aws" {
  region = "us-east-1"
}

# 1. Firewall: Open Port 80 for the Website and 22 for SSH
resource "aws_security_group" "blog_sg" {
  name = "blog-app-sg"
  ingress { 
    from_port = 22; to_port = 22; protocol = "tcp"; cidr_blocks = ["0.0.0.0/0"] 
  }
  ingress { 
    from_port = 80; to_port = 80; protocol = "tcp"; cidr_blocks = ["0.0.0.0/0"] 
  }
  egress { 
    from_port = 0; to_port = 0; protocol = "-1"; cidr_blocks = ["0.0.0.0/0"] 
  }
}

# 2. Server: Launch EC2 and Automate Software Installation
resource "aws_instance" "blog_server" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 (Verify for your region)
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.blog_sg.id]

  # This script runs on the first boot of the server
  user_data = <<-EOF
              #!/bin/bash
              sudo yum update -y
              sudo amazon-linux-extras install docker -y
              sudo service docker start
              sudo usermod -a -G docker ec2-user
              docker run -d -p 80:80 username/blog-app:latest
              EOF

  tags = { Name = "Live-Blog-Server" }
}

# 3. Output the IP so you can visit the site
output "website_url" { 
  value = aws_instance.blog_server.public_ip 
}
```
--------------------------------------------------------------------------
## Step 4: Execute the Deployment Lifecycle:
(a). Initialize Terraform:
terraform init
(b). Preview the plan:
terraform plan
(c). Apply the configuration:
terraform apply -auto-approve
(d). After a few minutes, Terraform will output the public IP of your EC2 instance. Copy that IP and paste it into your web browser to see your live blog application running on AWS!
--------------------------------------------------------------------------
## Step 5: Verification & Cleanup:
(a). Verify the deployment by visiting the public IP in your browser. You should see your blog application live.
(b). To avoid ongoing costs, destroy the infrastructure when done:
terraform destroy -auto-approve 