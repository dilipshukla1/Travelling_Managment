pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "dilipshukla1"
        IMAGE_NAME     = "travelling-php"
        IMAGE_TAG      = "latest"
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        SONAR_PROJECT  = "travelling-php"
    }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        /* Jenkins automatically checks out SCM
           DO NOT add checkout scm again */

        stage('SonarQube Code Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    docker run --rm \
                      -e SONAR_HOST_URL=$SONAR_HOST_URL \
                      -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
                      -v $WORKSPACE:/usr/src \
                      sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=${SONAR_PROJECT} \
                      -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                trivy fs --exit-code 1 --severity HIGH,CRITICAL .
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${FULL_IMAGE} .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image --exit-code 1 --severity HIGH,CRITICAL ${FULL_IMAGE}
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
            echo "✅ CI/CD pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed — check logs"
        }
    }
}
