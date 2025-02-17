pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'  
        ECR_REPO = 'public.ecr.aws/o8q1x1q3/mpm/appparalela'  
        TASK_FAMILY = 'paralelatask'  
        CLUSTER_NAME = 'paralelacluster'  // Nombre del cluster en ECS
        CONTAINER_NAME = 'mi-contenedor'  // Nombre del contenedor en la tarea ECS
        NEXUS_URL = 'http://localhost:8082'  // Si Nexus está en tu máquina local
        NEXUS_REPO = 'docker-releases'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop', url: 'https://github.com/marielapichardo/AppMovilD.git'
            }
        }

       stage('Build & Push to Nexus') {
            steps {
                script {
                    bat """
                    docker build -t %NEXUS_URL%/%NEXUS_REPO%/appparalela:latest .
                    docker login -u admin -p admin123 %NEXUS_URL%
                    docker push %NEXUS_URL%/%NEXUS_REPO%/appparalela:latest
                    """

                }
            }
        }


       stage('Pull & Push to AWS ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-credenciales', region: "${AWS_REGION}") {
                        sh """
                        docker pull ${NEXUS_URL}/${NEXUS_REPO}/appparalela:latest
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker tag ${NEXUS_URL}/${NEXUS_REPO}/appparalela:latest ${ECR_REPO}:latest
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
