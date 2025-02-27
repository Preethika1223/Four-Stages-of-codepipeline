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


