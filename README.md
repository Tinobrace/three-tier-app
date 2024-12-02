# Three-Tier Application with Docker, Docker Compose, and Nginx

This project demonstrates a simple three-tier architecture deployed using Docker and Docker Compose on an AWS EC2 instance. The architecture consists of:

- **Frontend**: React application
- **Backend**: Node.js API
- **Database**: MySQL

## Features

- Dockerized frontend, backend, and database services.
- Deployment using Docker Compose.
- Nginx as a reverse proxy to expose the frontend.

## Prerequisites

- AWS account with a free-tier EC2 instance.
- Basic understanding of Docker, Docker Compose, and Nginx.
- SSH access to the EC2 instance.

## Project Setup

### 1. **Clone the Repository**

```bash
git clone <repository-url>
cd <repository-folder>
```

### 2. **Frontend**

Navigate to the `frontend` directory and review the React app:
```bash
cd frontend
```

**Dockerfile for Frontend**:
```dockerfile
FROM node:16
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### 3. **Backend**

Navigate to the `backend` directory and review the Node.js server:
```bash
cd backend
```

**Dockerfile for Backend**:
```dockerfile
FROM node:16
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["node", "server.js"]
```

### 4. **Database**

Uses a standard MySQL image:
```yaml
db:
  image: mysql:5.7
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: myapp
```

### 5. **Docker Compose**

In the project root, the `docker-compose.yml` file orchestrates the services:
```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "4000:4000"
    depends_on:
      - db

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### 6. **EC2 Setup**

1. **Launch an EC2 Instance**:
   - Use an Ubuntu AMI (free-tier eligible).
   - Ensure the security group allows HTTP (port 80), and SSH (port 22).

2. **Install Docker and Docker Compose**:
   ```bash
   sudo apt update
   sudo apt install -y docker.io
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Install Nginx**:
   ```bash
   sudo apt install -y nginx
   ```

### 7. **Deploy the Application**

1. Run `docker-compose` to deploy all services:
   ```bash
   sudo docker-compose up -d
   ```

2. Verify running containers:
   ```bash
   sudo docker ps
   ```

### 8. **Nginx Configuration**

Edit the default Nginx config:
```bash
sudo nano /etc/nginx/sites-available/default
```

Set up reverse proxy for the frontend:
```nginx
server {
    listen 80;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Restart Nginx:
```bash
sudo systemctl restart nginx
```

## Access the Application

- **Frontend**: Visit `http://<EC2-public-IP>`.
- **Backend**: Verify connectivity from the frontend.
- **Database**: Pre-configured to store data for the backend.

## Conclusion

This project showcases how to deploy a three-tier architecture using Docker, Docker Compose, and Nginx on AWS EC2.

MIT License.

---

Feel free to modify this `README.md` to suit your project.
