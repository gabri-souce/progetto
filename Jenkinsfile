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
                # Installa Helm
                curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                chmod 700 get_helm.sh
                ./get_helm.sh
                
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
                    --set image.tag=latest
                '''
            }
        }
    }
}
