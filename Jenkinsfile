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

        stage('Deploy Helm Chart') {
            steps {
                script {
                    // Usa il kubeconfig di Kind otel-lab
                    withCredentials([file(credentialsId: 'kind-otel-lab-config', variable: 'KUBECONFIG_FILE')]) {
                        sh '''
                        # Imposta il kubeconfig per Kind
                        export KUBECONFIG=${KUBECONFIG_FILE}
                        
                        # Verifica il contesto (dovrebbe essere kind-otel-lab)
                        echo "🎯 Contesto Kubernetes in uso:"
                        kubectl config current-context
                        
                        echo "🖥️  Verifica nodi del cluster:"
                        kubectl get nodes
                        
                        echo "📦 Verifica namespaces:"
                        kubectl get namespaces
                        
                        # Verifica che Helm funzioni
                        helm version
                        
                        # Deploy dell'applicazione con timeout
                        helm upgrade --install flask-app ./helm/flask-app \
                            --set image.repository=gabrisource/otel-lab-app-python \
                            --set image.tag=latest \
                            --namespace default \
                            --create-namespace \
                            --wait \
                            --timeout 300s
                        
                        echo "✅ Deploy completato!"
                        echo "📋 Verifica pod:"
                        kubectl get pods -n default
                        '''
                    }
                }
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
