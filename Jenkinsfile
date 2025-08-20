pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '549328952286'
        IMAGE_REPO_NAME = 'demo-app'
        ECS_CLUSTER_NAME = 'demo-cluster'
        ECS_SERVICE_NAME = 'demo-service'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Abhish7899/demo-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest .'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com'
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    sh 'docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest'
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    IMAGE="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest"

                    NEW_TASK_DEF="""{
                        "family": "demo-task",
                        "networkMode": "awsvpc",
                        "requiresCompatibilities": ["FARGATE"],
                        "cpu": "256",
                        "memory": "512",
                        "executionRoleArn": "arn:aws:iam::$AWS_ACCOUNT_ID:role/ecsTaskExecutionRole",
                        "containerDefinitions": [
                            {
                                "name": "demo-app",
                                "image": "$IMAGE",
                                "cpu": 256,
                                "memory": 512,
                                "essential": true,
                                "portMappings": [
                                    {
                                        "containerPort": 80,
                                        "hostPort": 80,
                                        "protocol": "tcp"
                                    }
                                ],
                                "logConfiguration": {
                                    "logDriver": "awslogs",
                                    "options": {
                                        "awslogs-group": "/ecs/demo-task",
                                        "awslogs-region": "$AWS_DEFAULT_REGION",
                                        "awslogs-stream-prefix": "ecs"
                                    }
                                }
                            }
                        ]
                    }"""

                    // Register new task definition
                    sh """echo '$NEW_TASK_DEF' > taskdef.json"""
                    sh "aws ecs register-task-definition --cli-input-json file://taskdef.json"

                    // Update ECS service
                    sh "aws ecs update-service --cluster $ECS_CLUSTER_NAME --service $ECS_SERVICE_NAME --force-new-deployment"
                }
            }
        }
    }
}
