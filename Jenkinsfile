
pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '771805192968'
        AWS_DEFAULT_REGION = 'eu-north-1'
        ECR_REPO_NAME = 'devops-demo-app'
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Abhish7899/devops-projects-1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "🔹 Building Docker image..."
                    docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                    echo "🔹 Logging into AWS ECR..."
                    aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag & Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                    echo "🔹 Tagging and pushing image to ECR..."
                    docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS Cluster') {
            steps {
                script {
                    sh '''
                    echo "🔹 Updating kubeconfig for EKS..."
                    aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name devops-eks-cluster

                    echo "🔹 Deploying app to EKS..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    echo "🔹 Checking app status..."
                    kubectl get pods
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful — app should now be running on EKS!"
        }
        failure {
            echo "❌ Deployment failed — check Jenkins console logs for details."
        }
    }
}

