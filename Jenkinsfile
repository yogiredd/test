pipeline {
    agent any

    environment {
        IMAGE_NAME = "redhataccount/myapp"
        IMAGE_TAG = "5"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Git checkout successful'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building ${IMAGE_NAME}:${IMAGE_TAG}"
                    podman build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Testing image"
                    podman images
                '''
            }
        }

        stage('Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | podman login docker.io \
                            --username "$DOCKERHUB_USER" \
                            --password-stdin

                        podman tag ${IMAGE_NAME}:${IMAGE_TAG} \
                            docker.io/${IMAGE_NAME}:${IMAGE_TAG}

                        podman push docker.io/${IMAGE_NAME}:${IMAGE_TAG}

                        podman logout docker.io
                    '''
                }
            }
        }
    }
}
        
          
                
