 CI/CD Guide: GitHub Actions + Docker
------------------------------------------------------------
This updated guide focuses on the Docker-First approach, which is the industry standard for professional DevOps workflows.
It removes the manual Node.js installation (Part 1 of your previous notes) to keep your environment clean and reproducible.

Phase 1: Infrastructure Setup (AWS)
------------------------------------------------------------
Launch EC2 Instance:
AMI: Ubuntu 22.04 LTS.
Type: t2.micro (Free Tier).
Security Group: Ensure the following ports are open:
22 (SSH): For remote access.
3000 (App): The port your Node.js application will use.
Key Pair: Download your .pem file and keep it secure.
------------------------------------------------------------
Phase 2: Server Preparation (Docker)
Connect to your EC2 instance via SSH and set up the Docker engine:

Install Docker:
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

Apply Permissions (Instantly):
sudo usermod -aG docker ubuntu
newgrp docker  # Applies group changes without needing to log out
Verify: Run docker --version to ensure it is installed correctly.
--------------------------------------------------------------
Phase 3: Project Configuration (Local & GitHub)
Dockerfile: Add this to the root of your project:

Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]

GitHub Secrets: Go to Settings > Secrets and variables > Actions and add:

EC2_HOST: Your Public IPv4 address.

EC2_USER: ubuntu.

EC2_SSH_KEY: The full content of your .pem file.
------------------------------------------------------------
Phase 4: CI/CD Workflow (The "Brain")
Create .github/workflows/docker-deploy.yml in your repository:

YAML
name: Docker Deployment

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            # 1. Setup project directory
            if [ ! -d ~/backend-docker ]; then
              git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git ~/backend-docker
            fi
            cd ~/backend-docker

            # 2. Update code and build
            git pull origin main
            docker build -t backend-app .

            # 3. Refresh container
            docker stop backend-container || true
            docker rm backend-container || true
            docker run -d -p 3000:3000 --name backend-container backend-app
            
            # 4. Storage Maintenance
            docker image prune -f  # Deletes old images to save EC2 disk space
--------------------------------------------------------------            
Phase 5: Verification
Trigger: git push origin main.

Monitor: Check the Actions tab in GitHub for a green checkmark.

Verify: Run docker ps on your EC2 to see the container running.

Access: Visit http://YOUR_EC2_PUBLIC_IP:3000.
-------------------------------------------------------------

🛠️ Troubleshooting Cheat Sheet
Permission Denied: Run sudo chmod 666 /var/run/docker.sock if Jenkins/Actions can't talk to Docker.

Site Won't Load: Double-check that Port 3000 is open in your AWS Security Group.

Disk Full: If the EC2 stops responding, run docker system prune -a to clear cached data.