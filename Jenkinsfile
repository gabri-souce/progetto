pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gabrisource/otel-lab-app-python:latest"
        // Usa il kubeconfig di default del sistema
        KUBECONFIG = "${env.HOME}/.kube/config"
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

        stage('Setup Kubernetes Context') {
            steps {
                sh '''
                # Verifica che kubectl e helm siano disponibili
                echo "📦 Tool disponibili:"
                which kubectl || echo "kubectl non trovato"
                which helm || echo "helm non trovato"
                
                # Imposta il contesto Kind corretto
                kubectl config use-context kind-otel-lab
                
                echo "🎯 Contesto attuale:"
                kubectl config current-context
                
                echo "🖥️  Nodi del cluster:"
                kubectl get nodes
                '''
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                sh '''
                # Deploy dell'applicazione usando il contesto già configurato
                helm upgrade --install flask-app ./helm/flask-app \
                    --set image.repository=gabrisource/otel-lab-app-python \
                    --set image.tag=latest \
                    --namespace default \
                    --create-namespace \
                    --wait \
                    --timeout 300s
                
                echo "✅ Deploy completato!"
                echo "📋 Pod in esecuzione:"
                kubectl get pods -n default -w --timeout=30s
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
