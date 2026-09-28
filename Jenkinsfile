pipeline {

    agent any

    environment {
        IMAGE = "redhataccount/mynginx:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    podman build -t $IMAGE .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | podman login docker.io \
                          -u "$DOCKER_USER" \
                          --password-stdin

                        podman push $IMAGE

                        podman logout docker.io
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl set image deployment/myapp \
                          myapp=$IMAGE

                        kubectl rollout status deployment/myapp

                        kubectl get pods
                        kubectl get svc
                    '''
                }
            }
        }
    }
}
