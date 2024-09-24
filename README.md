# Dockerfile-Examples

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

