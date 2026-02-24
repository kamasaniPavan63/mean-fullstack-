# MEAN Stack Application Deployment  
### Docker | Nginx Reverse Proxy | MongoDB | GitHub Actions | AWS EC2

---

## 📌 Project Description

This project demonstrates the deployment of a production-ready MEAN Stack (MongoDB, Express.js, Angular, Node.js) CRUD application.

The application is:
- Containerized using Docker
- Deployed on AWS EC2 (Ubuntu)
- Configured with Nginx as a reverse proxy
- Automated using GitHub Actions CI/CD
- Connected to MongoDB installed directly on the EC2 server

---

## 🏗️ Architecture Overview

User (Browser)  
→ Nginx (Port 80)  
→ Angular Frontend (Docker – Port 8090)  
→ Node.js Backend API (Docker – Port 8080)  
→ MongoDB (Installed on EC2 – Port 27017)

---

# 🚀 Step-by-Step Setup and Deployment Instructions

---

## Step 1: Launch AWS EC2 Instance

1. Launch an Ubuntu EC2 instance.
2. Configure Security Group:
   - Port 22 (SSH)
   - Port 80 (HTTP)

Note: Ports 8080 and 8090 should not be publicly exposed when using Nginx reverse proxy.

---

## Step 2: Install Docker

SSH into your EC2 instance and run:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

Logout and login again to apply Docker group permissions.

---

## Step 3: Install Docker Compose

```bash
sudo apt install docker-compose -y
docker-compose --version
```

---

## Step 4: Install MongoDB on EC2

```bash
sudo apt install mongodb -y
sudo systemctl start mongodb
sudo systemctl enable mongodb
```

Verify MongoDB status:

```bash
sudo systemctl status mongodb
```

MongoDB will run locally at:

```
mongodb://localhost:27017
```

---

## Step 5: Create Docker Compose File

Create a file named `docker-compose.yml`:

```yaml
version: '3.8'

services:

  backend:
    image: pavankamasani/mean-backend:latest
    container_name: backend
    restart: always
    ports:
      - "8080:8080"
    environment:
      - MONGO_URI=mongodb://<EC2_PUBLIC_IP>:27017/dd_db

  frontend:
    image: pavankamasani/mean-frontend:latest
    container_name: frontend
    restart: always
    ports:
      - "8090:80"
    depends_on:
      - backend
```

Deploy containers:

```bash
docker-compose pull
docker-compose up -d
```

---

## Step 6: Configure Nginx Reverse Proxy

Install Nginx:

```bash
sudo apt install nginx -y
```

Create configuration file:

```bash
sudo nano /etc/nginx/sites-available/mean-app
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name <EC2_PUBLIC_IP>;

    location / {
        proxy_pass http://localhost:8090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }

    location /api/ {
        proxy_pass http://localhost:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/mean-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Your application will now be available at:

```
http://<EC2_PUBLIC_IP>
```

---

## Step 7: Setup GitHub Actions CI/CD

Create workflow file:

`.github/workflows/main.yml`

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: [self-hosted, Linux, X64, mean]

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Login to Docker Hub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Backend
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/mean-backend:latest ./backend

      - name: Push Backend
        run: docker push ${{ secrets.DOCKER_USERNAME }}/mean-backend:latest

      - name: Build Frontend
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/mean-frontend:latest ./frontend

      - name: Push Frontend
        run: docker push ${{ secrets.DOCKER_USERNAME }}/mean-frontend:latest

      - name: Deploy
        run: |
          docker-compose pull
          docker-compose down
          docker-compose up -d
```

---

## Step 8: Configure GitHub Secrets

Go to:

Repository → Settings → Secrets → Actions

Add:
- DOCKER_USERNAME
- DOCKER_PASSWORD

---

# 🔁 Deployment Workflow

1. Push code to the `main` branch.
2. GitHub Actions triggers automatically.
3. Docker images are built.
4. Images are pushed to Docker Hub.
5. EC2 pulls latest images.
6. Containers restart with updated version.
7. Nginx continues routing traffic to updated containers.

---

# 🛠 Useful Commands

Check running containers:

```bash
docker ps
```

Check container logs:

```bash
docker logs backend
docker logs frontend
```

Restart services:

```bash
docker-compose down
docker-compose up -d
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

---

# 🔒 Production Recommendations

- Enable HTTPS using Certbot (Let’s Encrypt)
- Secure MongoDB with authentication
- Configure firewall (UFW)
- Use environment variable files for secrets
- Avoid exposing backend ports publicly

---

## 👨‍💻 Author

Kamasani Pavan  
DevOps | AWS | Docker | CI/CD | Cloud Deployment
