pipeline {

    agent any

    environment {

        IMAGE_NAME = "devops-portfolio"

        AWS_REGION = "ap-south-1"

        ECR_REPO = "864899867861.dkr.ecr.ap-south-1.amazonaws.com/devops-portfolio"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh 'ls -lrt'
            }
        }

        stage('SonarQube Scan') {
            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=devops-portfolio \
                    -Dsonar.sources=. \
                    '''
                }
            }
        }

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('ECR Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-creds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {

                    sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Push to ECR') {

            steps {

                sh '''
                docker tag \
                ${IMAGE_NAME}:${BUILD_NUMBER} \
                ${ECR_REPO}:${BUILD_NUMBER}

                docker push \
                ${ECR_REPO}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {

        success {
            echo 'Image Successfully Uploaded To ECR'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}