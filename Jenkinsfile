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
                        /usr/local/bin/kubectl \
                          --kubeconfig="$KUBECONFIG" \
                          get nodes

                        /usr/local/bin/kubectl \
                          --kubeconfig="$KUBECONFIG" \
                          set image deployment/myapp \
                          myapp=$IMAGE

                        /usr/local/bin/kubectl \
                          --kubeconfig="$KUBECONFIG" \
                          rollout status deployment/myapp

                        /usr/local/bin/kubectl \
                          --kubeconfig="$KUBECONFIG" \
                          get pods

                        /usr/local/bin/kubectl \
                          --kubeconfig="$KUBECONFIG" \
                          get svc
                    '''
                }
            }
        }
    }
}

