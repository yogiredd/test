pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'redhataccount/mynginx'
        KUBECTL      = '/usr/local/bin/kubectl'
        DEPLOYMENT   = 'myapp'
        CONTAINER    = 'myapp'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                echo "Building image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"

                sh '''
                    podman build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo 'Pushing image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | podman login docker.io \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        podman push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        podman logout docker.io
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes...'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    sh '''
                        echo "Checking Kubernetes connection..."

                        ${KUBECTL} --kubeconfig="$KUBECONFIG" get nodes

                        echo "Updating deployment image..."

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            set image deployment/${DEPLOYMENT} \
                            ${CONTAINER}=${DOCKER_IMAGE}:${BUILD_NUMBER}

                        echo "Waiting for rollout..."

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            rollout status deployment/${DEPLOYMENT} \
                            --timeout=5m
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    sh '''
                        echo "Deployment status:"
                        ${KUBECTL} --kubeconfig="$KUBECONFIG" get deployment ${DEPLOYMENT}

                        echo "Pod status:"
                        ${KUBECTL} --kubeconfig="$KUBECONFIG" get pods -o wide

                        echo "Service status:"
                        ${KUBECTL} --kubeconfig="$KUBECONFIG" get svc
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "========================================"
            echo "       CI/CD PIPELINE SUCCESSFUL"
            echo "========================================"
            echo "Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo "Deployment: ${DEPLOYMENT}"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "         CI/CD PIPELINE FAILED"
            echo "========================================"
            echo "Please check the failed stage above."
            echo "========================================"
        }

        always {
            echo "Pipeline completed: Build #${BUILD_NUMBER}"
        }
    }
}

