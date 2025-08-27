pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gabri-souce/otel-lab-app-python:latest"
    }

    stages {
        stage('Clone repo') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/progetto.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // CONTESTO CORRETTO: usa la cartella otel-lab/app come contesto
                    sh 'docker build -t gabri-souce/otel-lab-app-python:latest -f otel-lab/app/Dockerfile ./otel-lab/app'
                }
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

        stage('Deploy Helm Chart') {
            steps {
                sh '''
                helm upgrade --install flask-app ./helm/flask-app \
                    --set image.repository=gabri-souce/otel-lab-app-python \
                    --set image.tag=latest
                '''
            }
        }
    }
}
