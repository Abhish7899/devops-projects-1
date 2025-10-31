pipeline {
    agent any

    environment {
        AWS_REGION = "eu-north-1"
        ECR_REPO = "devops-demo-app"
        IMAGE_TAG = "v6"
        AWS_ACCOUNT_ID = "771805192968"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "🔹 Checking out source code from Git..."
                git branch: 'main', url: 'https://github.com/Abhish7899/devops-projects-1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🔹 Building Docker image..."
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
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

        stage('Ensure ECR Repository Exists') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    script {
                        echo "🔹 Checking if ECR repository exists..."
                        sh '''
                            set -e
                            aws ecr describe-repositories --repository-names ${ECR_REPO} --region ${AWS_REGION} || \
                            aws ecr create-repository --repository-name ${ECR_REPO} --region ${AWS_REGION} \
                                --image-scanning-configuration scanOnPush=true \
                                --encryption-configuration encryptionType=AES256
                        '''
                    }
                }
            }
        }

        stage('Tag & Push Docker Image to ECR') {
            steps {
                script {
                    echo "🔹 Tagging and pushing Docker image to ECR..."
                    sh '''
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                        docker push ${ECR_URI}:${IMAGE_TAG}
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
                            aws eks update-kubeconfig --region ${AWS_REGION} --name devops-eks-cluster

                            # Replace image name in deployment manifest dynamically
                            sed -i "s|image:.*|image: ${ECR_URI}:${IMAGE_TAG}|" k8s/deployment.yaml

                            kubectl apply -f k8s/deployment.yaml
                            kubectl apply -f k8s/service.yaml

                            echo "✅ Deployment successful!"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Deployment failed — check Jenkins logs for details."
        }
    }
}
