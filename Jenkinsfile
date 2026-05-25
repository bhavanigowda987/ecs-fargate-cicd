pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '347152106940'
        ECR_REPO = 'myapp'
        ECS_CLUSTER = 'myCluster-dev'
        ECS_SERVICE = 'myService-dev'
        IMAGE_TAG = "jenkins-${BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bhavanigowda987/ecs-fargate-cicd.git'
            }
        }

        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_URI .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_URI'
            }
        }

        stage('Create Task Definition') {
            steps {
                sh '''
                cat > taskdef-jenkins.json <<TASK
{
  "family": "myTaskDef-dev",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::$AWS_ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "$IMAGE_URI",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
TASK
                '''
            }
        }

        stage('Register Task Definition') {
            steps {
                sh 'aws ecs register-task-definition --cli-input-json file://taskdef-jenkins.json --region $AWS_REGION'
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh 'aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition myTaskDef-dev --force-new-deployment --region $AWS_REGION'
            }
        }
    }
}
