# Containerization-of-a-react-django-app
Containerizing a React-Django application can streamline development, deployment, and scaling processes. This guide walks through the steps I took to containerize a React-Django application, addressing common issues along the way.

![flowchart of containerization process](https://github.com/user-attachments/assets/3651e558-2731-4807-8aea-992612cd3bda)

## Prerequisites
- Linux server from linode
- Docker and Docker-compose installed 
- A React-Django application from[﻿ github](https://github.com/Mrkaykab/React-Django-App.git)﻿


## 1. System Update
First, ensure your system is up to date:

```bash
sudo apt update && sudo apt upgrade -y
```
## 2. Install Git
If Git is not already installed:

```bash
sudo apt install git -y
```
## 3. Clone the Repository
Clone the React-Django-App repository:

```bash
git clone https://github.com/Mrkaykab/React-Django-App.git
cd React-Django-App/ComputexFrontend
```
![git clone](https://github.com/user-attachments/assets/7cbe5af0-3c68-40ff-8eb5-772580c12857)

## 4. Install Docker
Install Docker by following these steps:

```
sudo apt install docker.io -y
sudo apt install docker-compose
```
## 5. Create Dockerfile for frontend
create Dockerfile in `React-Django-App/ComputexFrontend` directory:

```
Nano Dockerfile
```
```
# Use an official Node.js runtime as a parent image
FROM node:16-alpine

# Set the working directory in the container
WORKDIR /app

# Copy the package.json and package-lock.json into the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code into the container
COPY . .

# Build the app for production
RUN npm run build

# Expose the port that the app will run on
EXPOSE 80

# Start an HTTP server to serve the built React app
RUN npm install -g serve
CMD ["serve", "-s", "dist", "-l", "80"]
```
![dockerfile frontend](https://github.com/user-attachments/assets/a4019ef7-d620-43c4-bdb7-32c4428cad5c)

**Change the Base_URL to server ip**

```bash
cd React-Django-App/ComputexFrontend/src/components/api
nano auth.ts
```
**Build the Frontend Docker Image**:

```
docker build -t computex-frontend .
```
**Run Frontend Container**:

```bash
docker run -d -p 80:80 computex-frontend
```

![docker frontend process](https://github.com/user-attachments/assets/3b30932f-91d0-48e3-9f12-6499d96beb16)

 I can now access the react-django-app frontend on port 80 of my server.

 ![docker frontend running](https://github.com/user-attachments/assets/f60a759f-8c96-4177-848d-60bac4557ba4)


## 6. Create Dockerfile for backend
create Dockerfile in `React-Django-App/Computex` directory:

```
# Use the official Python image from Docker Hub
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    python3-dev \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy the requirements.txt into the container
COPY requirements.txt /app/

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt \
    && pip install psycopg2-binary

# Copy the start script into the container and set permissions
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh

# Copy the rest of the application code into the container
COPY . /app/

# Ensure the script has execute permissions (in case it's overwritten by the COPY . /app/ command)
RUN chmod +x /app/start.sh

# Expose the port that Gunicorn will run on
EXPOSE 8000

# Set environment variables
ENV DJANGO_SETTINGS_MODULE=Computex.settings
ENV PYTHONUNBUFFERED=1

# Start the server
CMD ["/bin/sh", "/app/start.sh"]
```
![dockerfile backend](https://github.com/user-attachments/assets/477e0ca3-f7f8-4234-ba78-0dff20aad6f9)


## Create start script
Create a `start.sh` file in `React-Django-App/Computex `directory:

```bash
#!/bin/bash
python manage.py collectstatic --noinput
python manage.py migrate
gunicorn --workers 3 --bind 0.0.0.0:8000 Computex.wsgi:application
```
Update your Django settings (`Computex/settings.py`):

```python
import os
import dj_database_url

# ... other settings ...

DATABASES = {
    'default': dj_database_url.config(
        default=os.environ.get('DATABASE_URL'),
        conn_max_age=600
    )
}

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```
## Create Docker Compose file
Create a `docker-compose.yml` file in the `React-Django-App/Computex`directory:

```yaml
version: '3'
services:
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=computex
      - POSTGRES_USER=computexuser
      - POSTGRES_PASSWORD=computexpassword
  web:
    build: .
    command: ["/bin/sh", "/app/start.sh"]
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://computexuser:computexpassword@db:5432/computex
volumes:
  postgres_data:
```
![docker compose yml](https://github.com/user-attachments/assets/0e2566e6-00a4-49f3-819a-586142a790ee)


## Update requirements.txt
Ensure your `requirements.txt` file includes:

```
dj-database-url==1.3.0
gunicorn==20.1.0
```
##  Build and Run the Containers
```bash
docker-compose up --build
```
![docker compose ps](https://github.com/user-attachments/assets/6a4a9a1d-66ba-4d10-8e24-5eaf6bec8e1c)

![django backend working](https://github.com/user-attachments/assets/baadc925-2e42-46fd-b00d-bb32e6025a17)

## 7.  Test the Application
Once the containers started running, I was able to access my react application at `http://myserverip:8000` register a new account, then login to the account.

![reg_logged in to react app](https://github.com/user-attachments/assets/8e779fc4-7125-48c3-a7a4-fac8de89dab4)


## Conclusion
I have now successfully set up my system, cloned the React-Django-App repository, installed Docker and Docker Compose, and containerized the application. This setup provides a consistent environment for development and makes deployment easier.

