PROBLEM ASSIGNMENT 1:
To successfully complete the project of containerizing and deploying the Wisecow application on a Kubernetes environment with secure TLS communication, follow these steps:

### Dockerization

1. **Develop a Dockerfile**:
   - Create a Dockerfile in the root directory of the Wisecow application repository.
   - The Dockerfile should contain instructions to:
     - Set up the base image (e.g., an official image of the programming language or framework used).
     - Copy the application source code into the image.
     - Install any necessary dependencies.
     - Expose the necessary port(s).
     - Define the command to run the application.

   Example Dockerfile:
   ```Dockerfile
   FROM python:3.8-slim

   WORKDIR /app

   COPY . /app

   RUN pip install --no-cache-dir -r requirements.txt

   EXPOSE 8000

   CMD ["python", "app.py"]
   ```

### Kubernetes Deployment

2. **Craft Kubernetes Manifest Files**:
   - Create a deployment YAML file to define the application's deployment on Kubernetes, including replicas, image, and environment variables.
   - Create a service YAML file to expose the application, typically using a `LoadBalancer` or `NodePort` service type.
   - Optionally, create a ConfigMap or Secret YAML file to manage configuration data and sensitive information.

   Example deployment YAML:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: wisecow-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: wisecow
     template:
       metadata:
         labels:
           app: wisecow
       spec:
         containers:
         - name: wisecow
           image: <your-registry>/wisecow:latest
           ports:
           - containerPort: 8000
   ```

   Example service YAML:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: wisecow-service
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 8000
     selector:
       app: wisecow
   ```

### CI/CD Pipeline

3. **GitHub Actions Workflow**:
   - Create a GitHub Actions workflow file (`.github/workflows/deploy.yml`) to automate the build and push process.
   - Define steps to:
     - Check out the code.
     - Build the Docker image.
     - Push the image to a container registry (e.g., Docker Hub, AWS ECR).
     - Deploy the updated application to the Kubernetes environment using `kubectl`.

   Example GitHub Actions workflow:
   ```yaml
   name: CI/CD

   on:
     push:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
       - name: Check out code
         uses: actions/checkout@v2

       - name: Build Docker image
         run: docker build -t <your-registry>/wisecow:${{ github.sha }} .

       - name: Log in to registry
         run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

       - name: Push Docker image
         run: docker push <your-registry>/wisecow:${{ github.sha }}

     deploy:
       runs-on: ubuntu-latest
       needs: build
       steps:
       - name: Set up Kubernetes
         uses: Azure/setup-kubectl@v1
         with:
           version: '1.18.0'
           kubectl-version: '1.18.0'

       - name: Deploy to Kubernetes
         run: |
           kubectl set image deployment/wisecow-deployment wisecow=<your-registry>/wisecow:${{ github.sha }}
   ```

### TLS Implementation

4. **Secure TLS Communication**:
   - Use Kubernetes Ingress with TLS to secure the application.
   - Create an Ingress resource with TLS configuration, including the secret containing the TLS certificate and key.
   - The Ingress resource should route traffic to the service.

   Example Ingress YAML:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: wisecow-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     tls:
     - hosts:
       - wisecow.example.com
       secretName: wisecow-tls
     rules:
     - host: wisecow.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: wisecow-service
               port:
                 number: 80
   ```

### Expected Artifacts

- The GitHub repository should contain:
  - The source code of the Wisecow application.
  - The Dockerfile for the application.
  - Kubernetes manifest files (`deployment.yaml`, `service.yaml`, `ingress.yaml`, etc.).
  - GitHub Actions workflow file for CI/CD (`.github/workflows/deploy.yml`).

### Access Control

- Ensure the GitHub repository is set to public for accessibility.

### End Goal

- Successfully containerize and deploy the Wisecow application to a Kubernetes environment.
- Implement an automated CI/CD pipeline using GitHub Actions.
- Ensure the application is accessible via a secure HTTPS connection using TLS.

PROBLEM ASSIGNMENT 2:

 **System Health Monitoring Script** and **Automated Backup Solution**. Below are Bash scripts for both objectives.

### 1. System Health Monitoring Script

This script monitors CPU usage, memory usage, and disk space. It logs alerts if any metric exceeds predefined thresholds.

```bash
#!/bin/bash

# Thresholds
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80
LOG_FILE="/var/log/system_health.log"

# Function to log messages
log_message() {
    echo "$(date): $1" >> $LOG_FILE
}

# Check CPU usage
CPU_USAGE=$(top -b -n1 | grep "Cpu(s)" | awk '{print $2 + $4}')
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
    log_message "High CPU usage detected: ${CPU_USAGE}%"
fi

# Check Memory usage
MEMORY_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
if (( $(echo "$MEMORY_USAGE > $MEMORY_THRESHOLD" | bc -l) )); then
    log_message "High Memory usage detected: ${MEMORY_USAGE}%"
fi

# Check Disk usage
DISK_USAGE=$(df / | grep / | awk '{ print $5}' | sed 's/%//g')
if [ "$DISK_USAGE" -gt "$DISK_THRESHOLD" ]; then
    log_message "High Disk usage detected: ${DISK_USAGE}%"
fi

# Running processes count
PROCESS_COUNT=$(ps aux | wc -l)
log_message "Number of running processes: $PROCESS_COUNT"
```

### 2. Automated Backup Solution

This script backs up a specified directory to a remote server using `rsync`. It logs the success or failure of the backup operation.

```bash
#!/bin/bash

# Variables
SOURCE_DIR="/path/to/source"
DESTINATION="user@remote_server:/path/to/destination"
LOG_FILE="/var/log/backup.log"

# Perform backup using rsync
rsync -avz $SOURCE_DIR $DESTINATION &>> $LOG_FILE
if [ $? -eq 0 ]; then
    echo "$(date): Backup successful" >> $LOG_FILE
else
    echo "$(date): Backup failed" >> $LOG_FILE
fi
```

### Explanation

#### System Health Monitoring Script

1. **CPU Usage**: The script uses the `top` command to get the CPU usage and checks if it exceeds the threshold.
2. **Memory Usage**: It uses the `free` command to calculate memory usage.
3. **Disk Usage**: The `df` command is used to check the disk space usage.
4. **Logging**: Alerts are logged to a specified log file when thresholds are exceeded.

#### Automated Backup Solution

1. **Variables**: Define the source directory and destination (remote server).
2. **Rsync Command**: Uses `rsync` to perform the backup. The `-avz` options ensure the archive mode, verbose output, and compression during transfer.
3. **Logging**: Logs the success or failure of the backup operation to a log file.

These scripts provide basic functionality and can be extended or modified based on specific requirements, such as adding more detailed error handling, using environment variables for configuration, or integrating with monitoring and alerting systems.
