pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'eu-north-1'
        AWS_CREDENTIALS = credentials('aws-creds')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Abhish7899/devops-projects-1.git'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    withAWS(credentials: 'aws-creds', region: "${AWS_DEFAULT_REGION}") {
                        sh '''
                            echo "🔹 Initializing Terraform..."
                            terraform init
                            echo "🔹 Applying Terraform..."
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    sh '''
                        echo "🔹 Logging into Amazon ECR..."
                        aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin 771805192968.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com

                        echo "🔹 Building Docker image..."
                        docker build -t devops-app .

                        echo "🔹 Tagging image for ECR..."
                        docker tag devops-app:latest 771805192968.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/devops-app:latest

                        echo "🔹 Pushing image to ECR..."
                        docker push 771805192968.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/devops-app:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                        echo "🔹 Updating kubeconfig for EKS..."
                        aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name devops-eks-cluster

                        echo "🔹 Deploying application to EKS..."
                        kubectl apply -f k8s/

                        echo "🔹 Checking pods and services..."
                        kubectl get pods
                        kubectl get svc
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed successfully — check AWS Console for resources and EKS app deployment."
        }
    }
}


