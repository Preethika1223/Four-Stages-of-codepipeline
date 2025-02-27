# Four-Stages-of-codepipeline


---

# Jenkins-SonarQube-Docker CI/CD Pipeline

This project demonstrates a **CI/CD pipeline** using **Jenkins**, **SonarQube**, and **Docker**. The pipeline automates the process of code integration, quality analysis, and deployment.

---

## **Overview**
The pipeline performs the following steps:
1. **Jenkins** pulls code changes from a **GitHub** repository.
2. **SonarQube** analyzes the code for quality and security issues.
3. **Docker** builds a container image and deploys the application to a server.

---

## **Prerequisites**
1. **AWS EC2 Instances**:
   - Jenkins Server (Ports: `8080`, `22`)
   - SonarQube Server (Ports: `9000`, `22`)
   - Docker Server (Ports: `80`, `22`, `4243`)
2. **GitHub Repository** with a webhook configured.
3. **Docker Hub Account** for storing Docker images.

---

## **Setup Instructions**

### **1. Jenkins Setup**
- Install Jenkins on an EC2 instance:
  ```bash
  sudo apt update
  sudo apt install openjdk-17-jre -y
  sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt-get update
  sudo apt-get install jenkins -y
  sudo systemctl start jenkins
  sudo systemctl enable jenkins
  ```
- Access Jenkins at `http://<Jenkins-EC2-IP>:8080`.
- Configure GitHub webhook to trigger Jenkins builds.

---

### **2. SonarQube Setup**
- Install SonarQube on an EC2 instance:
  ```bash
  sudo apt update
  sudo apt install openjdk-17-jre -y
  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.1.0.102122.zip
  sudo apt install unzip -y
  unzip sonarqube-*.zip
  sudo mv sonarqube-* /opt/sonarqube
  cd /opt/sonarqube/bin/linux-x86-64/
  ./sonar.sh start
  ```
- Access SonarQube at `http://<SonarQube-EC2-IP>:9000`.
- Generate a SonarQube token for Jenkins integration.

---

### **3. Docker Setup**
- Install Docker on an EC2 instance:
  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl -y
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io -y
  sudo systemctl enable docker
  sudo systemctl start docker
  sudo usermod -aG docker ubuntu
  ```
- Configure Docker to listen on TCP:
  ```bash
  sudo vi /lib/systemd/system/docker.service
  ```
  Add the following line:
  ```bash
  ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
  ```
  Restart Docker:
  ```bash
  sudo systemctl daemon-reload
  sudo service docker restart
  ```

---

### **4. Jenkins Pipeline Configuration**
1. Install required plugins in Jenkins:
   - SonarQube Scanner
   - Docker
   - SSH
2. Configure SonarQube server and scanner in Jenkins.
3. Add a build step for SonarQube analysis:
   ```groovy
   node {
     stage('SCM') {
       checkout scm
     }
     stage('SonarQube Analysis') {
       def scannerHome = tool 'SonarScanner';
       withSonarQubeEnv() {
         sh "${scannerHome}/bin/sonar-scanner"
       }
     }
   }
   ```
4. Add a Docker build and publish step in the pipeline.
5. Add an SSH command to deploy the Docker container to the Docker EC2 instance:
   ```bash
   ssh -o StrictHostKeyChecking=no ubuntu@<Docker-EC2-IP> "
     echo 'Pulling latest Docker image...';
     docker pull <dockerhub-repo>/<image>:latest;
     echo 'Stopping and removing old container...';
     docker stop my-website || true;
     docker rm my-website || true;
     echo 'Starting new container...';
     docker run -d -p 80:80 --name my-website <dockerhub-repo>/<image>:latest;
     echo 'Deployment successful!';
   "
   ```

---

## **Dockerfile**
Here’s the Dockerfile used to build the application:
```Dockerfile
# Use the official Nginx image as a base
FROM nginx:1.25
# Copy the index.html file to the default Nginx web directory
COPY index.html /usr/share/nginx/html/
# Expose port 80
EXPOSE 80
# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

---

## **How to Use**
1. Push changes to the GitHub repository.
2. Jenkins will automatically trigger the pipeline.
3. SonarQube will analyze the code.
4. Docker will build and deploy the application.
5. Access the deployed application at `http://<Docker-EC2-IP>`.

---

## **Troubleshooting**
- Ensure all required ports are open in the security groups.
- Verify Jenkins, SonarQube, and Docker services are running.
- Check Jenkins logs for pipeline errors.

---

## **Contributing**
Feel free to contribute to this project by submitting issues or pull requests.

---

## **License**
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Let me know if you need further assistance!
