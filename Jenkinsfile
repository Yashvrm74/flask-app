pipeline { 
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Yashvrm74/flask-app.git'
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                sh 'python3 -m pip install --user virtualenv || true'
                sh 'python3 -m venv venv'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh './venv/bin/pip install --upgrade pip'
                sh './venv/bin/pip install -r requirements.txt'
            }
        }

        stage('Run Flask App') {
            steps {
              sh 'nohup ./venv/bin/python app.py > output.log 2>&1 &'
                
            }
        }

         stage('Build Docker Image') {
            steps {
              sh 'docker build -t flask-app:latest .'
                
            }
         }

         stage('Run Docker Container') {
            steps {
               sh 'docker rm -f flask-app || true'
               sh 'docker run -d --name flask-app -p 5001:5000 flask-app:latest'
            }
         }    
    }
}
