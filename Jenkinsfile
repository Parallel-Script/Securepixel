pipeline {
    agent any

    environment {
        ENV_VARS = credentials('django-env')  // Injects your secret .env
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Parallel-Script/Securepixel.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "$ENV_VARS" > SecurePixel/.env
                docker build -t securepixel-app .
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['app-ec2-ssh']) {
                  sh '''
                    mkdir -p ~/.ssh
                    ssh-keyscan -H 23.23.60.36 >> ~/.ssh/known_hosts

                    scp -o StrictHostKeyChecking=no docker-compose.yml nginx/default.conf ubuntu@23.23.60.36:/home/ubuntu/

                    ssh -o StrictHostKeyChecking=no ubuntu@23.23.60.36 '
                      docker stop django_app nginx_proxy || true &&
                      docker rm django_app nginx_proxy || true &&
                      docker-compose up -d --build
                    '
                  '''
                }                               
            }
        }
    }
}

