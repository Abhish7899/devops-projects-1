pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '549328952286'
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'demo-app'
        CLUSTER_NAME = 'crafty-deer-yk0kpi'
        SERVICE_NAME = 'demo-task-service-paynivjn'
        TASK_FAMILY = 'demo-task'
        CONTAINER_NAME = 'demo-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Abhish7899/demo-app.git'
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t $ECR_REPO .
                        docker tag $ECR_REPO:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                        docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                    '''
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh '''
                        # Create new task definition revision with latest image
                        TASK_DEF=$(aws ecs describe-task-definition --task-definition $TASK_FAMILY)
                        NEW_TASK_DEF=$(echo $TASK_DEF | jq --arg IMAGE "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest" '.taskDefinition | .containerDefinitions[0].image=$IMAGE | {family: .family, containerDefinitions: .containerDefinitions}')

                        echo $NEW_TASK_DEF > new-task-def.json
                        aws ecs register-task-definition --cli-input-json file://new-task-def.json

                        # Update ECS service to use new task definition
                        aws ecs update-service \
                            --cluster $CLUSTER_NAME \
                            --service $SERVICE_NAME \
                            --task-definition $TASK_FAMILY
                    '''
                }
            }
        }
    }
}

