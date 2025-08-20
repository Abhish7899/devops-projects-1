pipeline {
  agent any

  environment {
    AWS_ACCOUNT_ID = '549328952286'
    AWS_REGION     = 'us-east-1'
    ECR_REPO       = 'demo-app'
    CLUSTER_NAME   = 'crafty-deer-yk0kpi'
    SERVICE_NAME   = 'demo-task-service-paynivjn'
    TASK_FAMILY    = 'demo-task'
    CONTAINER_NAME = 'demo-app'
    IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/Abhish7899/demo-app.git'
      }
    }

    stage('Build Docker image') {
      steps {
        sh '''
          echo "Logging into Amazon ECR..."
          aws ecr get-login-password --region ${AWS_REGION} \
            | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

          echo "Building Docker image..."
          docker build -t ${ECR_REPO}:${BUILD_NUMBER} .

          echo "Tagging Docker image..."
          docker tag ${ECR_REPO}:${BUILD_NUMBER} ${IMAGE_URI}
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          echo "Pushing image to ECR..."
          docker push ${IMAGE_URI}
        '''
      }
    }

    stage('Deploy to ECS') {
      steps {
        sh '''
          echo "Fetching current task definition JSON..."
          aws ecs describe-task-definition \
            --task-definition ${TASK_FAMILY} \
            --region ${AWS_REGION} \
            --query 'taskDefinition' --output json > td.json

          echo "Producing a new revision with updated image..."
          cat td.json | jq \
            'del(.taskDefinitionArn,.revision,.status,.requiresAttributes,.registeredAt,.registeredBy,.compatibilities)
             | .containerDefinitions = (.containerDefinitions | map(
                 if .name=="'"${CONTAINER_NAME}"'" then .image="'"${IMAGE_URI}"'" else . end
               ))' > new-td.json

          echo "Registering new task definition..."
          NEW_TD_ARN=$(aws ecs register-task-definition \
            --cli-input-json file://new-td.json \
            --region ${AWS_REGION} \
            --query 'taskDefinition.taskDefinitionArn' --output text)

          echo "Updating service to new task definition: $NEW_TD_ARN"
          aws ecs update-service \
            --cluster ${CLUSTER_NAME} \
            --service ${SERVICE_NAME} \
            --task-definition $NEW_TD_ARN \
            --region ${AWS_REGION}
        '''
      }
    }
  }
}



