pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '992382778976'
        IMAGE_NAME = 'stats-dashboard'
        IMAGE_TAG = "${BUILD_ID}"
        DEPLOYMENT_KEY = '/var/lib/jenkins/.ssh/Berryman.pem'
        DEPLOYMENT_USER = 'ec2-user'
        DEPLOYMENT_HOST = '3.84.53.134'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '========== Checking out code =========='
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '========== Installing dependencies =========='
                sh '''
                    npm install
                '''
            }
        }

        stage('Build') {
            steps {
                echo '========== Building React app =========='
                sh '''
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                echo '========== Running tests =========='
                sh '''
                    CI=true npm test -- --coverage --watchAll=false || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '========== Building Docker image =========='
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo '========== Pushing image to AWS ECR =========='
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME}:latest ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:latest
                    docker push ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '========== Deploying to EC2 instance =========='
                sh '''
                    ssh -i ${DEPLOYMENT_KEY} ${DEPLOYMENT_USER}@${DEPLOYMENT_HOST} "docker stop stats-dashboard || true"
                    ssh -i ${DEPLOYMENT_KEY} ${DEPLOYMENT_USER}@${DEPLOYMENT_HOST} "docker rm stats-dashboard || true"
                    ssh -i ${DEPLOYMENT_KEY} ${DEPLOYMENT_USER}@${DEPLOYMENT_HOST} "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    ssh -i ${DEPLOYMENT_KEY} ${DEPLOYMENT_USER}@${DEPLOYMENT_HOST} "docker run -d --name stats-dashboard -p 80:80 ${ECR_REGISTRY}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}"
                '''
            }
        }
    }

    post {
        always {
            echo '========== Pipeline execution completed =========='
        }
        success {
            echo '========== Pipeline succeeded! =========='
        }
        failure {
            echo '========== Pipeline failed! =========='
        }
    }
}
