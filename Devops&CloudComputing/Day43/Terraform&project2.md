# Automated CI/CD Deployment:(GitHub + Docker + Terraform + AWS EC2)
------------------------------------------------------------------
We will follow "Zero-Touch" strategy. You will write code in VS Code, push it to GitHub, and let the automation handle the rest.

# Step 1: Local Preparation (VS Code)

Before automating, ensure your project files are ready in your workspace.

# Dockerfile: 
This file defines how to build your application image. It should be in the root of your project.

```Dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```
# Step 2: Build and Push Docker Image to Docker Hub
(a). Build your image:
- if not already logged in: docker login
     Verify docker is installed and running by executing: docker --version   
     And then run :docker ps
- If no errors appear → Docker is working.
    docker build -t username/app-name .
    Ex-docker build -t devopslearner07/terraform-frontend-project .
- Now verify the image is created by running: docker images
(b)Push to Docker Hub (ensure the repository is set to Public):
 - docker push username/app-name
 - Ex-docker push devopslearner07/terraform-frontend-project
(c): Local Verification
- Run the container locally to ensure it works: docker run -d -p 9090:80 devopslearner07/terraform-frontend-project
Visit: http://localhost:9090 to see your app running locally.
-------------------------------------------------------------------------------------
# Step 3: Configure GitHub Automation

- To achieve "Zero-Touch," GitHub needs your credentials stored securely.

# (a). Create GitHub Secrets:
  Go to your Repo → Settings → Secrets and variables → Actions. Add:

 1. AWS_ACCESS_KEY_ID: your AWS Access Key ID

 2. AWS_SECRET_ACCESS_KEY: your AWS Secret Access Key

 3. DOCKER_USERNAME: (Your Docker Hub ID)

 4. DOCKER_PASSWORD: (Your Docker Hub Password)

 # (b). Create Workflow File:

 -Create a folder .github/workflows/ and add deploy.yml:

 ```yaml
name: DevOps Pipeline

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build & Push
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/terraform-frontend-project:latest .
          docker push ${{ secrets.DOCKER_USERNAME }}/terraform-frontend-project:latest

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init & Apply
        run: |
          terraform init
          terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

```
------------------------------------------------------------------------------
# Step 4: Write Terraform Configuration:
(a). Create a file named main.tf in a separate folder. This code acts as the blueprint for your cloud hardware.

```hcl

provider "aws" {
  region = "us-east-1"
}

# 1. Security Group: Fixed syntax (removed semicolons)
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

# 2. Server: Launch EC2 and Automate Deployment
resource "aws_instance" "blog_server" {
  ami                    = "ami-0440d3b780d96b29d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.blog_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              # Update and Install Docker
              dnf update -y
              dnf install -y docker
              
              # Start and Enable Docker service
              systemctl start docker
              systemctl enable docker
              
              # Add default user to docker group
              usermod -aG docker ec2-user
              
              # Run the container from Docker Hub
              docker run -d -p 80:80 devopslearner07/terraform-frontend-project:latest
              EOF

  tags = {
    Name = "Live-Blog-Server"
  }
}

# 3. Output: Final website URL
output "website_url" {
  value = "http://${aws_instance.blog_server.public_ip}"
}
```
---------------------------------------------------------------------------------
# step 5: Push to Deploy
- Now, commit and push your code to GitHub. The workflow will trigger automatically, building your Docker image, pushing it to Docker Hub, and deploying it on AWS EC2 using Terraform.

```
git init
git add .
git commit -m "Deploying via GitHub Actions"
git push origin main
```
- After a few minutes, check the Actions tab in your GitHub repo to see the workflow progress. Once completed, you can access your app using the public IP provided in the Terraform output.
---------------------------------------------------------------------------------
# step 6: Cleanup (Destroy)

- To avoid unnecessary costs, remember to destroy your infrastructure when done:

- Initialize then Destroy:
 Since you are using a container, you need to run the init command first using the same Docker syntax, and then run the destroy.
 docker run --rm -v ${PWD}:/workspace -w /workspace hashicorp/terraform:latest init

 Then run the destroy command:

```
docker run --rm -it `
  -e AWS_ACCESS_KEY_ID="secrets.AWS_ACCESS_KEY_ID" `
  -e AWS_SECRET_ACCESS_KEY="secrets.AWS_SECRET_ACCESS_KEY" `
  -v ${PWD}:/workspace -w /workspace `
  hashicorp/terraform:latest destroy -auto-approve
```