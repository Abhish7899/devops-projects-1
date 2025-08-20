pipeline {
  agent any
  environment {
    AWS_REGION     = 'us-east-1'                 
    AWS_ACCOUNT_ID = '549328952286'          
    ECR_REPO       = 'demo-app'
    IMAGE_TAG      = "${env.BUILD_NUMBER}"        
    IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"

    CLUSTER_NAME   = 'crafty-deer-yk0kpi'
    SERVICE_NAME   = 'demo-task-service-paynivjn'
    TASK_FAMILY    = 'demo-task'              
    CONTAINER_NAME = 'demo-app'                   
  }
  options { timestamps() }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Docker build') {
      steps {
        sh """
          aws --version
          echo Logging in to ECR...
          aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

          # Ensure repo exists (safe if already exists)
          aws ecr describe-repositories --repository-names ${ECR_REPO} --region ${AWS_REGION} >/dev/null 2>&1 || \
            aws ecr create-repository --repository-name ${ECR_REPO} --region ${AWS_REGION}

          echo Building image...
          docker build -t ${ECR_REPO}:${IMAGE_TAG} .

          docker tag ${ECR_REPO}:${IMAGE_TAG} ${IMAGE_URI}
          docker push ${IMAGE_URI}
        """
      }
    }
    stage('Deploy to ECS') {
      steps {
        sh """
          echo "Fetching current task definition JSON..."
          aws ecs describe-task-definition \
            --task-definition ${TASK_FAMILY} \
            --region ${AWS_REGION} \
            --query 'taskDefinition' --output json > td.json

          echo "Producing a new revision with updated image..."
          # Remove fields not allowed on register and set new image
          cat td.json | jq '
            del(.taskDefinitionArn,.revision,.status,.requiresAttributes,.registeredAt,.registeredBy,.compatibilities)
            | .containerDefinitions = (.containerDefinitions | map(
                if .name=="${CONTAINER_NAME}" then .image="${IMAGE_URI}" else . end
              ))
          ' > new-td.json

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
        """
      }
    }
  }
  post {
    success {
      echo "Deployed ${IMAGE_URI} to ${SERVICE_NAME} on ${CLUSTER_NAME}"
    }
  }
}

