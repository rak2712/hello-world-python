pipeline {
    agent any

    environment {
        IMAGE_NAME = "rakshith3/hello-world-python:latest"   // Use your actual Docker Hub repo name
        CONTAINER_NAME = "hello-world-flask"
        PORT = "5005"
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                // Clone the GitHub repo (main branch)
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
                echo "📦 Logging in and pushing image to Docker Hub..."
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

        stage('Deploy Container (Optional)') {
            steps {
                echo "🚀 Running container locally (for test)..."
                sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME --restart unless-stopped -p $PORT:5005 $IMAGE_NAME
                '''
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
