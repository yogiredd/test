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
                echo 'Push stage will be configured next'
            }
        }
    }
}

       
                
