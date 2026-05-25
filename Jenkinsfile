pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '347152106940'
        ECR_REPO = 'myapp'
        ECS_CLUSTER = 'myCluster-dev'
        ECS_SERVICE = 'myService-dev'
        TASK_DEFINITION = 'myTaskDef-dev'
        IMAGE_TAG = "jenkins-${BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/bhavanigowda987/ecs-fargate-cicd.git'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecr get-login-password --region $AWS_REGION |
