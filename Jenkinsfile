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
                git 'https://github.com/rak2712/hello-world-python.git'
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
                    credentialsId: 'dockerhub-credentials', // This should match the ID in Jenkins Credentials
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
            cleanWs() // Cleans workspace after every build (success or fail)
        }
    }
}
