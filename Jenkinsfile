pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gabrisource/otel-lab-app-python:latest"
        KUBECONFIG = "/root/.kube/config" // kubeconfig valido dentro Jenkins
    }

    stages {
        stage('Clone repo') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/progetto.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} -f app/Dockerfile ./app'
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

        stage('Setup Kubernetes Context') {
            steps {
                sh '''
                echo "📦 Verifica strumenti:"
                which kubectl || echo "kubectl non trovato"
                which helm || echo "helm non trovato"

                echo "🎯 Configuro cluster Kind"
                kubectl config set-cluster kind-otel-lab --server=https://172.19.0.2:6443 --insecure-skip-tls-verify=true
                kubectl config use-context kind-otel-lab

                echo "🎯 Contesto attuale:"
                kubectl config current-context

                echo "🖥️  Nodi del cluster:"
                kubectl get nodes
                '''
            }
        }

        stage('Create ImagePullSecret (if needed)') {
            steps {
                sh '''
                # Solo se il repository Docker è privato
                kubectl get secret regcred -n default || \
                kubectl create secret docker-registry regcred \
                    --docker-server=https://index.docker.io/v1/ \
                    --docker-username=<USERNAME> \
                    --docker-password=<PASSWORD> \
                    --docker-email=<EMAIL> \
                    -n default
                '''
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                sh '''
                echo "🚀 Deploy Helm Chart"
                helm upgrade --install flask-app ./helm/flask-app \
                    --set image.repository=gabrisource/otel-lab-app-python \
                    --set image.tag=latest \
                    --namespace default \
                    --create-namespace \
                    --wait \
                    --timeout 300s

                echo "📋 Verifico lo stato dei pod:"
                kubectl wait --for=condition=Ready pod -l app=flask-app -n default --timeout=120s
                kubectl get pods -n default
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completata con successo!"
        }
        failure {
            echo "❌ Pipeline fallita. Controlla i log per dettagli."
        }
    }
}

