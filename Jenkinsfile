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

        stage('Explore Repository Structure') {
            steps {
                sh '''
                echo "=== STRUTTURA COMPLETA DEL REPOSITORY ==="
                ls -la
                echo ""
                echo "=== CONTENUTO DELLE SOTTOCARTELLE ==="
                find . -type d -name "*app*" -o -name "*otel*" -o -name "*docker*" | head -20
                echo ""
                echo "=== CERCO DOCKERFILE ==="
                find . -name "Dockerfile" -type f
                echo ""
                echo "=== CERCO REQUIREMENTS.TXT ==="
                find . -name "requirements.txt" -type f
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Prima verifichiamo la struttura, poi costruiamo
                    def dockerfilePath = sh(script: 'find . -name "Dockerfile" -type f | head -1', returnStdout: true).trim()
                    def contextPath = dockerfilePath.replace('/Dockerfile', '')
                    
                    echo "Trovato Dockerfile in: ${dockerfilePath}"
                    echo "Context path: ${contextPath}"
                    
                    sh "docker build -t gabri-souce/otel-lab-app-python:latest -f ${dockerfilePath} ${contextPath}"
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
