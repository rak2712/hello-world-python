pipeline {
    agent any

    environment {
        IMAGE_NAME = "rak2712/hello-world-python:latest"
        CONTAINER_NAME = "hello-world-flask"
        PORT = "5005"
    }

    stages {

        stage('Clone Repo') {
            steps {
                // Explicitly specify 'main' branch to avoid defaulting to 'master'
                git branch: 'main', url: 'https://github.com/rak2712/hello-world-python.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials', // Must match ID in Jenkins Credentials
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                        docker logout
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME -p $PORT:5005 $IMAGE_NAME
                """
            }
        }
    }

    post {
        always {
            cleanWs() // Always clean up workspace to avoid leftover files
        }
    }
}
