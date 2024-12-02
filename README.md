# three-tier-app
Project Setup

1. Clone the Repository
git clone <repository-url>
cd <repository-folder>

2. Frontend
Navigate to the frontend directory and review the React app:
cd frontend

Dockerfile for Frontend:
FROM node:16
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

3. Backend
Navigate to the backend directory and review the Node.js server:
cd backend

Dockerfile for Backend:
FROM node:16
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["node", "server.js"]

4. Database
Uses a standard MySQL image:
db:
  image: mysql:5.7
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: myapp
   
6. Docker Compose
In the project root, the docker-compose.yml file orchestrates the services:
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
  
6. EC2 Setup
Launch an EC2 Instance:
Use an Ubuntu AMI (free-tier eligible).
Ensure the security group allows HTTP (port 80), and SSH (port 22).

Install Docker and Docker Compose:
sudo apt update
sudo apt install -y docker.io
sudo curl -L "https://github.com/docker/compose/releases/download/v2.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

Install Nginx:
sudo apt install -y nginx

7. Deploy the Application
Run docker-compose to deploy all services:
sudo docker-compose up -d

Verify running containers:
sudo docker ps

8. Nginx Configuration
Edit the default Nginx config:
sudo nano /etc/nginx/sites-available/default

Set up reverse proxy for the frontend:
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

Restart Nginx:
sudo systemctl restart nginx
