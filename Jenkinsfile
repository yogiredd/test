```groovy
pipeline {

    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

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

        stage('Trivy Security Scan') {
            steps {
                echo '========================================'
                echo 'Running Trivy Security Scan'
                echo '========================================'

                sh '''
                    IMAGE_TAR="/tmp/mynginx-${BUILD_NUMBER}.tar"

                    echo "Saving Podman image..."

                    podman save \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -o "$IMAGE_TAR"

                    echo "Scanning image with Trivy..."

                    trivy image \
                        --input "$IMAGE_TAR" \
                        --severity HIGH,CRITICAL \
                        --exit-code 0

                    echo "Removing temporary image archive..."

                    rm -f "$IMAGE_TAR"
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

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            get nodes

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

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            get deployment ${DEPLOYMENT}

                        echo "Pod status:"

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            get pods -o wide

                        echo "Service status:"

                        ${KUBECTL} \
                            --kubeconfig="$KUBECONFIG" \
                            get svc
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
            echo "Trivy: Security scan completed"
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
```
