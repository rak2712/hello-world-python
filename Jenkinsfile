pipeline {
    agent any

    environment {
        IMAGE_NAME = "rakshith3/hello-world-python:latest"
        SERVICE_PORT = "5005"
        NODE_PORT = "30005"
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/rak2712/hello-world-python.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🔧 Building Docker image: $IMAGE_NAME"
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                echo "📦 Pushing image to Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                        docker logout
                    '''
                }
            }
        }

        stage('Test Kubernetes Access') {
            steps {
                echo "🔍 Verifying Kubernetes connection..."
                sh 'kubectl get pods -n default'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes..."
                sh 'kubectl apply -f k8s/deployment.yaml --validate=false'
            }
        }

        stage('Port-Forward Service for External Access') {
            steps {
                echo "🌐 Starting port-forward to expose service on all interfaces..."
                // Kill existing port-forward if any, then start new one in background
                sh '''
                    pkill -f "kubectl port-forward svc/hello-python-service $SERVICE_PORT:$SERVICE_PORT" || true
                    nohup kubectl port-forward svc/hello-python-service $SERVICE_PORT:$SERVICE_PORT --address 0.0.0.0 > port-forward.log 2>&1 &
                '''
            }
        }

        stage('Show Access Info') {
            steps {
                script {
                    // Get host IP (simplified)
                    def hostIp = sh(script: "hostname -I | awk '{print \$1}'", returnStdout: true).trim()
                    echo "✅ Your app is now accessible at:"
                    echo "    http://${hostIp}:${SERVICE_PORT}"
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning workspace"
            cleanWs()
        }
    }
}
