A **step-by-step guide** to build the four stages of pipeline.

---

## **Step 1: Create 3 EC2 Instances**
1. **Log in to AWS Console** and navigate to EC2.
2. **Launch 3 Spot Instances** with the following details:
   - **AMI**: Ubuntu 22.04 LTS
   - **Instance Type**: `t2.medium`
   - **Storage**: 16 GB
   - **Security Group**: `launch-wizard-1` (ensure ports `8080`, `9000`, `80`, and `22` are open).
   - **Key Pair**: Use your existing key pair.
   - **Instance Names**:
     - Jenkins: `ID_jenkins`
     - SonarQube: `ID_SonarQube`
     - Docker: `ID_Dockers`

---

## **Step 2: Install Jenkins**
1. **Connect to Jenkins EC2** using **Instance Connect**.
2. **Install Jenkins**:
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
3. **Access Jenkins**:
   - Open `http://<Jenkins-EC2-IP>:8080` in your browser.
   - Unlock Jenkins using the initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Complete the setup and create an admin user (`admin/admin`).

---

## **Step 3: Connect Jenkins to GitHub**
1. **Create a GitHub Webhook**:
   - Go to your GitHub repository → Settings → Webhooks → Add webhook.
   - Payload URL: `http://<Jenkins-EC2-IP>:8080/github-webhook/`
   - Content type: `application/json`
   - Trigger on: `Push` and `Pull request` events.
2. **Create a Jenkins Pipeline**:
   - In Jenkins, create a new `Freestyle project` (e.g., `IDpipeline`).
   - Under `Source Code Management`, select `Git` and provide your repository URL.
   - Under `Build Triggers`, select `GitHub hook trigger for GITScm polling`.
   - Save the project.

---

## **Step 4: Install SonarQube**
1. **Connect to SonarQube EC2** using **Instance Connect**.
2. **Install SonarQube**:
   ```bash
   sudo apt update
   sudo apt install openjdk-17-jre -y
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.1.0.102122.zip
   sudo apt install unzip -y
   unzip sonarqube-*.zip
   sudo mv sonarqube-* /opt/sonarqube
   cd /opt/sonarqube/bin/linux-x86-64/
   ./sonar.sh start
   ./sonar.sh console
   ```
3. **Access SonarQube**:
   - Open `http://<SonarQube-EC2-IP>:9000` in your browser.
   - Login with `admin/admin` and change the password.
4. **Create a SonarQube Project**:
   - Name: `ID`
   - Select `Global repositories` → `GitHub` → `Others`.
   - Note down the `Project Key` (e.g., `sonar.projectKey=Id`).
5. **Generate a SonarQube Token**:
   - Go to `My Account → Security → Generate Token`.
   - Name: `Jenkins`
   - Type: `Global`
   - Duration: `30 days`
   - Note down the token (e.g., `sqa_0bd83a93e687e1cd1cfc1025e48fe51dff9d1b12`).

---

## **Step 5: Configure SonarQube in Jenkins**
1. **Install Plugins**:
   - Go to `Manage Jenkins → Plugins` and install:
     - SonarQube Scanner
     - SSH2easy
     - SSHServer
     - Docker
     - CloudBees Docker Build and Publish
     - SSH Agent
2. **Configure SonarQube Scanner**:
   - Go to `Manage Jenkins → Tools` and add the SonarQube Scanner.
3. **Add SonarQube Server**:
   - Go to `Manage Jenkins → System` and add SonarQube server details:
     - Name: `SonarQube`
     - URL: `http://<SonarQube-EC2-IP>:9000`
     - Authentication: Use the SonarQube token generated earlier.
4. **Update Jenkins Pipeline**:
   - Go to your pipeline (`IDpipeline`) → Configure → Build Steps.
   - Add a build step: `Execute SonarQube Scanner`.
   - Paste the `Project Key` under `Analysis Properties`:
     ```bash
     sonar.projectKey=Id
     ```

---

## **Step 6: Install Docker**
1. **Connect to Docker EC2** using **Instance Connect**.
2. **Install Docker**:
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
   sudo usermod -aG docker jenkins
   ```
3. **Configure Docker to Listen on TCP**:
   - Edit `/lib/systemd/system/docker.service`:
     ```bash
     ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
     ```
   - Restart Docker:
     ```bash
     sudo systemctl daemon-reload
     sudo service docker restart
     ```

---

## **Step 7: Integrate Docker with Jenkins**

1. **Connect to Jenkins EC2** via connect instance:
2. **Install Docker** (if not already installed):
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
   ```
3. **Test Docker Installation**:
   ```bash
   sudo docker run hello-world
   sudo docker ps
   sudo docker --version
   ```

---

### **Step 2: Add Jenkins User to Docker Group**
1. Run the following commands to add the `ubuntu`, `$USER`, and `jenkins` users to the `docker` group:
   ```bash
   sudo usermod -aG docker ubuntu
   sudo usermod -aG docker $USER
   sudo usermod -aG docker jenkins
   ```
2. **Restart Docker and Jenkins**:
   ```bash
   sudo systemctl restart docker
   sudo systemctl status docker
   sudo systemctl restart jenkins
   ```
3. **Verify Jenkins User Can Access Docker**:
   ```bash
   sudo -u jenkins docker login -u <dockerhub-username>
   ```
   - Enter your Docker Hub password when prompted.

---

### **Step 3: Configure Docker in Jenkins**
1. **Go to Jenkins Website**:
   - Open `http://<Jenkins-EC2-IP>:8080` in your browser.
2. **Configure Docker Installation**:
   - Go to `Manage Jenkins → Tools`.
   - Under `Docker Installations`, click `Add Docker`.
   - Name: `docker`
   - Version: `latest`
   - Check `Install automatically`.
   - Click `Save`.
3. **Add Docker Build Step to Pipeline**:
   - Go to your pipeline (e.g., `A10001pipeline`) → Configure.
   - Under `Build Steps`, click `Add build step` and select `Docker Build and Publish`.
   - Configure the following:
     - **Repository Name**: `<dockerhub-username>/<repository-name>` (e.g., `xyz/a10001`).
     - **Tag**: `latest`
     - **Registry Credentials**: Click `Add` and provide your Docker Hub username and password.
     - Select the credentials from the dropdown.
4. **Advanced Docker Configuration**:
   - Click the `Advanced` dropdown.
   - Ensure the following options are checked:
     - **Force Pull**: Yes
     - **Create Fingerprints**: Yes
   - Under `Docker Installation`, select `docker` (the name configured earlier).
5. **Add Execute Shell Command**:
   - Scroll down to the `Execute Shell` section.
   - Paste the following command to deploy the Docker container to the Docker EC2 instance:
     ```bash
     ssh -o StrictHostKeyChecking=no ubuntu@<Docker-EC2-IP> "
       echo 'Pulling latest Docker image...';
       docker pull <dockerhub-username>/<repository-name>:latest;
       echo 'Stopping and removing old container...';
       docker stop my-website || true;
       docker rm my-website || true;
       echo 'Starting new container...';
       docker run -d -p 80:80 --name my-website <dockerhub-username>/<repository-name>:latest;
       echo 'Deployment successful!';
     "
     ```
   - Replace `<Docker-EC2-IP>`, `<dockerhub-username>`, and `<repository-name>` with your actual values.

---

### **Step 4: Add SSH Keys for Docker EC2**
1. **Go to Jenkins Pipeline**:
   - Select your pipeline (e.g., `A10001pipeline`) → Configure.
2. **Add SSH Agent**:
   - Under `Environment`, check `SSH Agent`.
   - Click `Add` under `Credentials`.
   - Select `SSH Username with Private Key`.
   - Provide the following:
     - **ID**: Any unique identifier (e.g., `docker-ec2-key`).
     - **Description**: Optional description.
     - **Username**: `ubuntu` (the username for your Docker EC2 instance).
     - **Private Key**: Paste the contents of your `.pem` file (private key for Docker EC2).
   - Click `Save`.

---

### **Step 5: Test the Pipeline**
1. **Trigger the Pipeline**:
   - Make changes to your GitHub repository (e.g., update `index.html`).
   - Verify that the Jenkins pipeline triggers automatically.
2. **Check Deployment**:
   - Open `http://<Docker-EC2-IP>` in your browser to see the updated content.

---

## **Dockerfile**
Ensure your GitHub repository contains the following `Dockerfile`:
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
