pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        AWS_ACCOUNT_ID = '771805192968'
        ECR_REPO = 'devops-demo-app'
        IMAGE_TAG = "v${BUILD_NUMBER}"
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
                    echo "🔹 Building Docker image..."
                    sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    script {
                        echo "🔹 Logging into AWS ECR..."
                        sh '''
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        '''
                    }
                }
            }
        }

        stage('Tag & Push Docker Image to ECR') {
            steps {
                script {
                    echo "🔹 Tagging and pushing Docker image..."
                    sh '''
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    script {
                        echo "🔹 Deploying to EKS cluster..."
                        sh '''
                            aws eks update-kubeconfig --name devops-eks-cluster --region ${AWS_REGION}
                            kubectl set image deployment/devops-demo-app \
                                devops-demo-app=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                            kubectl rollout status deployment/devops-demo-app
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful — version ${IMAGE_TAG} is live on EKS!"
        }
        failure {
            echo "❌ Deployment failed — check Jenkins logs for details."
        }
    }
}



