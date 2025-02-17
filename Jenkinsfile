pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '<831926602540>.dkr.ecr.us-east-1.amazonaws.com/<mpm/appparalela>'
        TASK_FAMILY = 'paralelatask'
        CLUSTER_NAME = 'paralelacluster'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/marielapichardo/AppMovilD.git'
            }
        }
        
        stage('Build & Push Image') {
            steps {
                script {
                    withAWS(credentials: 'aws-credenciales', region: "${AWS_REGION}") {
                        sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker build -t ${ECR_REPO}:latest .
                        docker push ${ECR_REPO}:latest
                        """
                    }
                }
            }
        }

        stage('Update ECS Task') {
            steps {
                script {
                    withAWS(credentials: 'aws-credenciales', region: "${AWS_REGION}") {
                        sh """
                        aws ecs update-service --cluster ${CLUSTER_NAME} --service ${TASK_FAMILY} --force-new-deployment
                        """
                    }
                }
            }
        }
    }
}