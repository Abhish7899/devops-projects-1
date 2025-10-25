pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhishekdhurwade/demo-app:latest"
        KUBECONFIG = "/home/jenkins/.kube/config"   // adjust path if different
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Abhish7899/demo-app.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s/deployment.yaml --kubeconfig=$KUBECONFIG
                    kubectl rollout status deployment/demo-app --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }
    }
}



