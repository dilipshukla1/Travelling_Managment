pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "dilipshukla1"
        IMAGE_NAME     = "travelling-php"
        IMAGE_TAG      = "latest"
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${FULL_IMAGE} .
                '''
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                  docker push ${FULL_IMAGE}
                '''
            }
        }
    }

    post {
        success {
            echo "Docker image built and pushed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
