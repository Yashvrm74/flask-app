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
    }
}
