# Flask App with Jenkins CI/CD Pipeline and Docker Deployment

This project demonstrates a simple **Flask "Hello World" app** with a complete **CI/CD pipeline** using **Jenkins**, **Docker**, and a **GitHub Webhook**. It also includes a scheduled cleanup job for old containers.

---

## 🌐 Application Overview

A minimal Flask app that returns “Hello, World!” when accessed.

### `app.py`

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

---

## 📁 Project Structure

```
flask-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── Jenkinsfile
```

---

## 🧪 Requirements

- Ubuntu-based EC2 VM (or any Linux server)
- Jenkins installed
- Docker installed
- GitHub public repository

---

## ⚙️ Jenkins Setup

### 1. Install Jenkins on your VM

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /etc/apt/trusted.gpg.d/jenkins.asc
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
```

Access Jenkins via: `http://<your-ec2-ip>:8080`

### 2. Install Git & Docker on Jenkins VM

```bash
sudo apt install git -y
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
```

---

## 🔧 GitHub Webhook Configuration

1. Go to your GitHub repo → **Settings** → **Webhooks**
2. Add a new webhook:
   - Payload URL: `http://<your-ec2-ip>:8080/github-webhook/`
   - Content type: `application/json`
   - Trigger: Just the push event

Ensure Jenkins has the **"GitHub Integration Plugin"** installed.

---

## 📦 Dockerfile

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

## 🛠️ Jenkinsfile (CI/CD Pipeline)

```groovy
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/yashvrm74/flask-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt || true'
            }
        }

        stage('Run Flask App (Detached)') {
            steps {
                sh 'nohup python3 app.py &'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop flask-app || true
                docker rm flask-app || true
                docker run -d --name flask-app -p 5000:5000 flask-app:latest
                '''
            }
        }
    }
}
```

> Replace `yashvrm74` with your actual GitHub username if different.

---

## 🧹 Scheduled Cleanup Job

We use a shell script to remove **stopped containers older than 7 days** and **dangling images**.

### Script: `/usr/local/bin/remove_old_containers.sh`

```bash
#!/bin/bash

seven_days_ago=$(date --date="7 days ago" +%s)

for container in $(docker ps -a --filter "status=exited" --format "{{.ID}}"); do
    created=$(docker inspect --format '{{.Created}}' "$container")
    created_ts=$(date --date="$created" +%s)

    if [ "$created_ts" -lt "$seven_days_ago" ]; then
        echo "Removing container $container created on $created"
        docker rm "$container"
    fi
done

docker images -f "dangling=true" -q | xargs -r docker rmi
```

### Setup

```bash
sudo nano /usr/local/bin/remove_old_containers.sh
sudo chmod +x /usr/local/bin/remove_old_containers.sh
```

### Schedule in Crontab

```bash
sudo crontab -e
```

Add this line to run daily at midnight:

```bash
0 0 * * * /usr/local/bin/remove_old_containers.sh
```
