# 🚀 DevOps Pipeline for Vite.js Chat Application

This project demonstrates a complete CI/CD DevOps pipeline setup for a Vite.js-based frontend chat application. The backend is hosted on [Render](https://render.com), and the frontend is deployed through a Dockerized Jenkins pipeline using AWS EC2 instances.

# Images
![media](https://github.com/user-attachments/assets/993eb59b-f09d-4830-a6f8-5b748b8c8626)

![media](https://github.com/user-attachments/assets/01423928-301a-4dc6-9e09-43401f569140)

![media](https://github.com/user-attachments/assets/e33b305e-b24b-426d-bb7d-ec9083adf5c5)

![media](https://github.com/user-attachments/assets/227c1d73-14b8-4a9b-89c3-a8f249be2594)

![media](https://github.com/user-attachments/assets/be61ce2a-4a64-43de-a784-abee4b3ec7dc)

---

## 🧱 Project Structure

- **Frontend**: Vite.js React chat application
- **Backend**: Hosted on Render, connected via API URL
- **Pipeline**: Jenkins + Docker + AWS EC2 + GitHub Webhooks

---

## ⚙️ DevOps Pipeline Flow

### ✅ 1. Project Initialization

- Built a MERN stack chat app with **Vite.js** frontend.
- The backend is hosted on **Render** and connected to the frontend using the backend URL.
- Locally tested the frontend:
  ```bash
  docker build -t vite-chat-app .
  docker run -d -p 3000:3000 vite-chat-app
  ````

### 🗃️ 2. Version Control & Team Collaboration

* Created a GitHub repository: [devops](https://github.com/ArunSadalgekar07/devops)
* Pushed the frontend source code to this repo.
* Added 3 team members as collaborators.

---

### ☁️ 3. AWS Infrastructure

| Instance    | Purpose        | IP Type    | Description                    |
| ----------- | -------------- | ---------- | ------------------------------ |
| Jenkins EC2 | Jenkins server | Elastic IP | Hosts Jenkins CI tool          |
| Docker EC2  | App Deployment | Elastic IP | Builds & runs Docker container |

* Both instances had **Elastic IPs** assigned for stable addressing.
* Added **swap memory** to both instances for stability.
* Security Groups were configured to allow:

  * **Port 8080** for Jenkins access
  * **Port 3000** for the app

---

## 🔧 Jenkins Setup

* Installed Jenkins on the Jenkins EC2 instance.

* Configured:

  * **Swap Memory** for smoother operation
  * Required **plugins**: Git, Docker, SSH, Email Extension
  * Added **credentials**:

    * `ec2-ssh-key` for Docker instance access
    * GitHub token for repository access

* Created a **Pipeline job** named `final-chat`.

* Connected the pipeline to GitHub repo:

  ```
  https://github.com/ArunSadalgekar07/devops.git
  ```

---

## 🔗 Webhook Integration

* Configured GitHub webhook to Jenkins:

  ```
  http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
  ```
* This triggers Jenkins build on every `push` to the `main` branch.

---

## 📦 Jenkinsfile (CI/CD Pipeline)

The pipeline performs the following:

1. Clones the GitHub repo
2. Builds the Docker image on the Docker EC2 instance
3. Runs the container and exposes it on port 3000
4. Sends email notifications to team members on build success/failure
5. Placeholder for Selenium test stage (not used due to t2.micro limitations)

<details>
<summary>📄 Jenkinsfile</summary>

```groovy
pipeline {
    agent any

    environment {
        DOCKER_HOST_IP = "51.21.54.222"
        DOCKER_USER = "ubuntu"
        DOCKER_APP_DIR = "chat-app"
        RECIPIENTS = '02fe23bcs411@kletech.ac.in, 02fe23bcs430@kletech.ac.in, 02fe23bcs420@kletech.ac.in, 02fe22bcs050@kletech.ac.in'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ArunSadalgekar07/devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                        ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                            rm -rf ${DOCKER_APP_DIR} && mkdir -p ${DOCKER_APP_DIR}
                        '

                        scp -i \$KEY -o StrictHostKeyChecking=no -r \
                            src public \
                            Dockerfile package.json package-lock.json vite.config.js index.html\
                            ${DOCKER_USER}@${DOCKER_HOST_IP}:${DOCKER_APP_DIR}/

                        ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                            cd ${DOCKER_APP_DIR} &&
                            docker build -t vite-chat-app .
                        '
                    """
                }
            }
        }

        stage('Run Container') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                        ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                            docker rm -f vite-chat-container || true &&
                            docker run -d -p 3000:3000 --name vite-chat-container vite-chat-app
                        '
                    """
                }
            }
        }

        stage('Selenium Tests') {
            steps {
                sh """
                    echo "Running Selenium tests..."
                    # TODO: Add your Selenium test command here
                """
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ Build Successful: ${env.JOB_NAME} [#${env.BUILD_NUMBER}]",
                body: """Great news!

Build #${env.BUILD_NUMBER} of job '${env.JOB_NAME}' succeeded.

View it here: ${env.BUILD_URL}""",
                to: "${env.RECIPIENTS}"
            )
        }

        failure {
            emailext(
                subject: "❌ Build Failed: ${env.JOB_NAME} [#${env.BUILD_NUMBER}]",
                body: """Oops!

Build #${env.BUILD_NUMBER} of job '${env.JOB_NAME}' failed.

View it here: ${env.BUILD_URL}""",
                to: "${env.RECIPIENTS}"
            )
        }
    }
}
```

</details>

---

## ⚠️ Limitations

* Due to memory constraints of **t2.micro**, **Selenium tests** were **not executed**.
* App is currently in a **development-level deployment** without load balancing, monitoring, or auto-scaling.

---

## 🚀 Future Improvements

* ✅ Integrate **Selenium/Playwright** tests on larger EC2 instance
* 🌀 Implement **Blue-Green Deployment** for zero-downtime upgrades
* 📊 Add **Monitoring** using Prometheus + Grafana
* 🐳 Use **Amazon ECR** for managing Docker images

---

## 👨‍💻 Team Members

* Arun Sadalgekar 
* Yash Shinde
* Harsh Gadekar
* Manasi Ekbote

---

## 📁 Repository Overview

| File / Folder     | Description                           |
| ----------------- | ------------------------------------- |
| `Dockerfile`      | Docker instructions for Vite frontend |
| `Jenkinsfile`     | Jenkins CI/CD automation script       |
| `src/`, `public/` | React app source code                 |
| `vite.config.js`  | Vite.js configuration                 |
| `package.json`    | Project dependencies and scripts      |



> 💬 For any questions or improvements, feel free to open an issue or contact the contributors.



