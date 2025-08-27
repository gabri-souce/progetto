pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gabrisource/otel-lab-app-python:latest"
    }

    stages {
        stage('Clone repo') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/progetto.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t gabrisource/otel-lab-app-python:latest -f app/Dockerfile ./app'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Install Helm') {
            steps {
                sh '''
                # Installa Helm senza sudo
                curl -fsSL -o helm.tar.gz https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz
                tar -zxvf helm.tar.gz
                mv linux-amd64/helm /usr/local/bin/helm
                chmod +x /usr/local/bin/helm
                
                # Verifica l'installazione
                helm version
                '''
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                sh '''
                helm upgrade --install flask-app ./helm/flask-app \
                    --set image.repository=gabrisource/otel-lab-app-python \
                    --set image.tag=latest \
                    --namespace default \
                    --create-namespace
                '''
            }
        }
    }
}
