pipeline {
    agent any

    environment {
        IMAGE_NAME = "yogiredd/myapp"
        IMAGE_TAG = "4"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'podman build -t ${IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh 'podman images ${IMAGE}'
            }
        }

        stage('Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        podman login docker.io \
                          -u "$DOCKER_USER" \
                          -p "$DOCKER_PASS"

                        podman push \
                          ${IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }
}
