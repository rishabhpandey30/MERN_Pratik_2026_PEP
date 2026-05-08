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
- First install the AWS CLI tool if you haven't already: https://aws.amazon.com/cli/
- verify the version by running: aws --version
- Open your terminal and run: aws configure
- verify the access keys by running: aws sts get-caller-identity (if you have already configured the keys in AWS CLI)
- Enter your AWS Access Key ID and AWS Secret Access Key.
- Set the Default region name to us-east-1 (or your preferred region).
- Set the Default output format to json (or your preferred format).
--------------------------------------------------------------------------
## Step 2: Containerize the Project:
(a).Create a Dockerfile in your project's root folder with this content:

```Dockerfile

FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```
(b).Build your image:
- if not already logged in: docker login
Verify docker is installed and running by executing: docker --version
And then run :docker ps
- If no errors appear → Docker is working.

docker build -t username/app-name .

Ex-docker build -t devopslearner07/terraform-frontend-project .

- Now verify the image is created by running: docker images

(c)Push to Docker Hub (ensure the repository is set to Public):
docker push username/app-name

Ex-docker push devopslearner07/terraform-frontend-project

(d): Local Verification
- Run the container locally to ensure it works: docker run -d -p 9090:80 devopslearner07/terraform-frontend-project
Visit: http://localhost:9090 to see your app running locally.
--------------------------------------------------------------------------
## Step 3: Write Terraform Configuration:

(a). Create a file named main.tf in a separate folder. This code acts as the blueprint for your cloud hardware.

```hcl 

provider "aws" {
  region = "us-east-1"
}

# 1. Security Group: Open Port 80 for Web and 22 for SSH
resource "aws_security_group" "blog_sg" {
  name        = "blog-app-sg"
  description = "Allow HTTP and SSH traffic"

  ingress { 
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] 
  }

  ingress { 
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] 
  }

  egress { 
    from_port   = 0
    to_port     = 0
    protocol    = "-1" # Allows all outbound traffic
    cidr_blocks = ["0.0.0.0/0"] 
  }
}

# 2. Server: Launch EC2 and Automate Docker Setup
resource "aws_instance" "blog_server" {
  # Use Amazon Linux 2023 AMI (Update this ID based on current us-east-1 availability)
  ami                    = "ami-0440d3b780d96b29d" 
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.blog_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              # Update system
              dnf update -y
              # Install and start Docker
              dnf install -y docker
              systemctl start docker
              systemctl enable docker
              # Add user to docker group
              usermod -aG docker ec2-user
              # Run your specific container
              docker run -d -p 80:80 devopslearner07/terraform-frontend-project:latest
              EOF

  tags = { Name = "Live-Blog-Server" }
}

# 3. Output the IP with "http://" for easy clicking
output "website_url" { 
  value = "http://${aws_instance.blog_server.public_ip}" 
}
```
--------------------------------------------------------------------------
## Step 4:(a) Execute the Deployment Lifecycle:
(a). Initialize Terraform:
terraform init
(b). Preview the plan:
terraform plan
(c). Apply the configuration:
terraform apply -auto-approve
(d). After a few minutes, Terraform will output the public IP of your EC2 instance. Copy that IP and paste it into your web browser to see your live blog application running on AWS!

# (b).Run Terraform via Docker (Optional):
 # Initialize:
 This sets up the working directory inside a temporary container.
Run Command:
docker run --rm -v ${PWD}:/workspace -w /workspace hashicorp/terraform:latest init

2. Deploy (Apply)
This command passes your AWS keys into the container environment variables so it can talk to your AWS account. Replace the keys below with your actual values:

```
docker run --rm -it `
  -e AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY" `
  -e AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY" `
  -v ${PWD}:/workspace -w /workspace `
  hashicorp/terraform:latest apply

  ```

- Now Wait for the prompt: It will show you the plan. Type yes and hit Enter.
- The Result: Terraform will create the Security Group and the EC2 instance, then automatically run your Docker image on that server.
- Terraform will show you a list of things it will create (Security Group and EC2).
- It will ask: Do you want to perform these actions?
- Type yes and press Enter.
--------------------------------------------------------------------------
# Step 5: Verification
Once the command finishes, Terraform will output your website_url.
Copy the IP address.

Paste it into your browser: http://<YOUR_EC2_IP>.

**Important:** Point out that you didn't need to install anything but Docker to make this happen.
--------------------------------------------------------------------------
## Step 5: Verification & Cleanup:
(a). Verify the deployment by visiting the public IP in your browser. You should see your blog application live.
(b). To avoid ongoing costs, destroy the infrastructure when done: terraform destroy -auto-approve 

or if using Docker: 

```
docker run --rm -it `
  -e AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY" `
  -e AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY" `
  -v ${PWD}:/workspace -w /workspace `
  hashicorp/terraform:latest destroy -auto-approve
  ``` 

  (c). Clean Up All Stopped Containers

      docker container prune -f