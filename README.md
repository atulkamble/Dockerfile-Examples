# Dockerfile-Examples

Here's the suggested structure for your repository on Dockerfile examples:

Let me know if you'd like further refinements or additions to the repo structure!

Here's a step-by-step guide on creating `Dockerfile` examples and setting them up on an EC2 instance:

---

### **Setting Up Docker and Running Docker Containers on EC2**

#### **1. Launch an EC2 Instance**
1. Go to your AWS Management Console.
2. Launch a new EC2 instance using an Amazon Linux 2 or Ubuntu AMI.
3. Choose instance type (e.g., t2.micro).
4. Configure security groups:
   - Allow inbound traffic on port 22 for SSH access.
   - Allow port 5000 (for Flask) or port 3000 (for Node.js) for external access to your app.

#### **2. Connect to the EC2 Instance**

1. SSH into your EC2 instance:
   ```bash
   ssh -i your-key.pem ec2-user@<ec2-public-ip>
   ```

#### **3. Install Docker on EC2 Instance**

For **Amazon Linux 2**:

1. Update the package manager:
   ```bash
   sudo yum update -y
   ```

2. Install Docker:
   ```bash
   sudo amazon-linux-extras install docker -y
   ```

3. Start Docker service and enable it to run on startup:
   ```bash
   sudo service docker start
   sudo systemctl enable docker
   ```

4. Add the current user to the `docker` group:
   ```bash
   sudo usermod -a -G docker ec2-user
   ```

   (You may need to log out and log back in for this to take effect.)

For **Ubuntu**:

1. Update the package manager:
   ```bash
   sudo apt-get update
   ```

2. Install Docker:
   ```bash
   sudo apt-get install docker.io -y
   ```

3. Start Docker service:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

#### **4. Install Git and Clone Your Application Code**

1. Install Git:
   ```bash
   sudo yum install git -y   # Amazon Linux
   sudo apt-get install git -y  # Ubuntu
   ```

2. Clone your repository containing the Dockerfile and application code:
   ```bash
   git clone https://github.com/yourusername/your-repo.git
   cd your-repo
   ```
### **Topics Covered:**
1. **Basic Dockerfile Setup**:
   - Python (Flask)
   - Node.js (Express)
   - Java (Spring Boot)
   - PHP (Laravel)
   - .NET Core Application

2. **Advanced Dockerfile Techniques**:
   - Multi-stage builds for optimized image sizes
   - Caching strategies for faster builds
   - Running containers as non-root users for security
   - Health checks for container monitoring
   - Using build arguments and environment variables

3. **Docker Compose for Multi-Container Applications**:
   - Setting up multiple services (e.g., Nginx, Node.js, PostgreSQL) using `docker-compose.yml`

4. **Best Practices**:
   - Image size optimizations
   - Security considerations
   - Automated health checks
   - Auto-restarting containers on failure

---

#### **5. Build the Docker Image**

1. Build the Docker image from the `Dockerfile`:
   ```bash
   docker build -t your-app-image .
   ```

#### **6. Run the Docker Container**

1. Run the Docker container with port forwarding:
   - For Flask:
     ```bash
     docker run -d -p 5000:5000 your-app-image
     ```

   - For Node.js:
     ```bash
     docker run -d -p 3000:3000 your-app-image
     ```

2. The application will now be accessible via the EC2 public IP on the specified port:
   - Flask App: `http://<ec2-public-ip>:5000`
   - Node.js App: `http://<ec2-public-ip>:3000`

#### **7. (Optional) Auto-start Docker Container on Reboot**

To ensure the Docker container starts automatically when the EC2 instance reboots, create a `systemd` service for your Docker container.

1. Create a new service file:
   ```bash
   sudo nano /etc/systemd/system/myapp.service
   ```

2. Add the following content:
   ```ini
   [Unit]
   Description=Docker Container for MyApp
   After=docker.service
   Requires=docker.service

   [Service]
   Restart=always
   ExecStart=/usr/bin/docker start -a <container_name>
   ExecStop=/usr/bin/docker stop -t 2 <container_name>

   [Install]
   WantedBy=multi-user.target
   ```

3. Enable the service:
   ```bash
   sudo systemctl enable myapp
   ```

---

This setup will get your Docker-based applications running on an EC2 instance. Let me know if you need further customizations for any specific applications or use cases!


---

### **1. Dockerfile Example for a Python (Flask) App**

**Dockerfile:**

```Dockerfile
# Base image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose the port the app runs on
EXPOSE 5000

# Set the command to run the app
CMD ["python", "app.py"]
```

**Example Python App (Flask):**
- `app.py`
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, Flask on Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

- `requirements.txt`
```
Flask==2.1.2
```

### **2. Dockerfile Example for Node.js (Express) App**

**Dockerfile:**

```Dockerfile
# Base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install --production

# Copy application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Command to run the app
CMD ["node", "app.js"]
```

**Example Node.js App:**
- `app.js`
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Express on Docker!');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

- `package.json`
```json
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```
Here are examples of basic `Dockerfile` setups for different types of applications:

### 1. **Python Application (Flask)**

```Dockerfile
# Base image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 5000

# Set the command to run the app
CMD ["python", "app.py"]
```

### 2. **Node.js Application (Express)**

```Dockerfile
# Base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install --production

# Copy application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Command to run the application
CMD ["node", "app.js"]
```

### 3. **Java Application (Spring Boot)**

```Dockerfile
# Base image
FROM openjdk:17-jdk-slim

# Set the working directory
WORKDIR /app

# Copy the jar file
COPY target/myapp.jar /app/myapp.jar

# Expose port 8080
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

### 4. **PHP Application (Laravel)**

```Dockerfile
# Base image
FROM php:8.1-fpm-alpine

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN docker-php-ext-install pdo pdo_mysql

# Copy application code
COPY . .

# Install composer and dependencies
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
RUN composer install --no-dev --optimize-autoloader

# Expose port 9000 for PHP-FPM
EXPOSE 9000

# Start the PHP-FPM server
CMD ["php-fpm"]
```

### 5. **.NET Core Application**

```Dockerfile
# Base image for build
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80

# Copy the application files
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY . .

# Restore the dependencies and build the app
RUN dotnet restore "MyApp/MyApp.csproj"
RUN dotnet build "MyApp/MyApp.csproj" -c Release -o /app/build

# Publish the app
FROM build AS publish
RUN dotnet publish "MyApp/MyApp.csproj" -c Release -o /app/publish

# Create the final image
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

These Dockerfiles demonstrate basic setups. You can modify them based on specific needs such as adding environment variables, using multi-stage builds for optimization, or using custom configurations for the applications.

Here are some **advanced Dockerfile examples** covering various use cases, optimizations, and multi-stage builds for better performance, security, and flexibility.

---

### **1. Python Application with Multi-stage Build and Caching**

This Dockerfile demonstrates caching Python dependencies and using multi-stage builds for smaller final images.

```Dockerfile
# Stage 1: Build stage with dependencies
FROM python:3.9-slim AS builder

# Install pip and set up environment
RUN apt-get update && apt-get install -y build-essential
WORKDIR /app

# Copy only requirements to leverage Docker caching
COPY requirements.txt .

# Install dependencies
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Final stage, minimal runtime environment
FROM python:3.9-slim

# Set environment variables for Python
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONUNBUFFERED=1

# Copy dependencies from build stage
COPY --from=builder /root/.local /root/.local

# Copy the application code
COPY . /app
WORKDIR /app

# Expose the port Flask will run on
EXPOSE 5000

# Command to run the app
CMD ["python", "app.py"]
```

### **2. Node.js Application with Non-root User and Multi-stage Build**

This Dockerfile sets up a Node.js app and adds security by running the container as a non-root user.

```Dockerfile
# Stage 1: Build stage
FROM node:16-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy application source code
COPY . .

# Stage 2: Final stage with non-root user
FROM node:16-alpine

# Create non-root user
RUN addgroup -S nodegroup && adduser -S nodeuser -G nodegroup

# Set working directory
WORKDIR /app

# Copy built application code from builder stage
COPY --from=builder /app /app

# Change ownership of the application
RUN chown -R nodeuser:nodegroup /app

# Use non-root user
USER nodeuser

# Expose the port the app runs on
EXPOSE 3000

# Command to run the app
CMD ["node", "app.js"]
```

### **3. Java Application with Multi-stage Build (Maven and JAR)**

This Dockerfile demonstrates a multi-stage build for a Java Spring Boot application, leveraging Maven for dependency resolution and reducing the final image size.

```Dockerfile
# Stage 1: Build the application with Maven
FROM maven:3.8.4-openjdk-17 AS builder

# Set the working directory
WORKDIR /build

# Copy the pom.xml and resolve dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy the source code and build the JAR file
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Create the final image with JRE
FROM openjdk:17-jdk-slim

# Set the working directory
WORKDIR /app

# Copy the JAR file from the builder stage
COPY --from=builder /build/target/myapp.jar /app/myapp.jar

# Expose the application's port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

### **4. PHP Application (Laravel) with Optimized Apache Setup**

This Dockerfile sets up a Laravel application with Apache and includes optimizations for caching dependencies and permissions.

```Dockerfile
# Base image
FROM php:8.1-apache

# Enable required PHP extensions and Apache modules
RUN docker-php-ext-install pdo pdo_mysql && a2enmod rewrite

# Set the working directory
WORKDIR /var/www/html

# Install Composer (dependency manager for PHP)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Copy application code and install dependencies
COPY . /var/www/html
RUN composer install --optimize-autoloader --no-dev

# Set ownership and permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# Expose port 80 for Apache
EXPOSE 80

# Start Apache server
CMD ["apache2-foreground"]
```

### **5. Multi-container Docker Setup (Docker Compose)**

This example uses Docker Compose to manage multiple services (Nginx, Node.js, and a PostgreSQL database) within a single setup.

**Dockerfile for Node.js:**
```Dockerfile
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

**Docker Compose Configuration:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - nodejs

  nodejs:
    build: ./node-app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

### **6. Using Build Arguments and Environment Variables**

This Dockerfile makes use of build arguments (`ARG`) and environment variables (`ENV`), which allow configuration during build and runtime.

```Dockerfile
# Base image
FROM python:3.9-slim

# Build arguments (passed during docker build)
ARG ENVIRONMENT=production

# Environment variables (available during container runtime)
ENV ENVIRONMENT=${ENVIRONMENT}
ENV APP_PORT=5000

# Set the working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose the application port
EXPOSE ${APP_PORT}

# Run the app
CMD ["python", "app.py"]
```

You can build the image with:
```bash
docker build --build-arg ENVIRONMENT=staging -t my-app .
```

### **7. Using Health Checks in Dockerfile**

This Dockerfile adds a health check to ensure that the application inside the container is running and healthy.

```Dockerfile
FROM node:16-alpine

# Set the working directory
WORKDIR /app

# Copy and install dependencies
COPY package*.json ./
RUN npm install

# Copy application code
COPY . .

# Expose the application port
EXPOSE 3000

# Add a health check to ensure the app is responding
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl --fail http://localhost:3000 || exit 1

# Start the application
CMD ["npm", "start"]
```

---

### **Best Practices for Advanced Dockerfiles:**
1. **Use Multi-stage Builds:** Reduces image size by separating build dependencies from runtime dependencies.
2. **Use Non-root User:** Enhances security by avoiding running containers as root.
3. **Leverage Caching:** Optimize builds by leveraging Docker’s layer caching mechanism (e.g., copy `requirements.txt` first before copying the rest).
4. **Health Checks:** Monitor and restart unhealthy containers automatically.
5. **Use Build Arguments and Environment Variables:** Allow flexible configuration at both build and runtime.
6. **Minimal Base Images:** Choose lightweight base images (e.g., `alpine`) to reduce image size and attack surface.
7. **Clean Up:** Remove temporary files, packages, or build tools after installation to keep the final image clean.

These advanced Dockerfile techniques ensure your containerized applications are more efficient, secure, and production-ready.
