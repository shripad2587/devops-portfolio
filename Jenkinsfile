pipeline {

    agent any

    environment {

        IMAGE_NAME   = "devops-portfolio"
        AWS_REGION   = "ap-south-1"

        ECR_REGISTRY = "864899867861.dkr.ecr.ap-south-1.amazonaws.com"
        ECR_REPO     = "devops-portfolio"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                pwd
                ls -lrt
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {

                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonarqube') {

                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=devops-portfolio \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8
                        """
                    }
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
                    --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {

                sh '''
                docker tag \
                ${IMAGE_NAME}:${BUILD_NUMBER} \
                ${ECR_REGISTRY}/${ECR_REPO}:${BUILD_NUMBER}
                '''

            }
        }

        stage('Push To ECR') {
            steps {

                sh '''
                docker push \
                ${ECR_REGISTRY}/${ECR_REPO}:${BUILD_NUMBER}
                '''

            }
        }

        stage('Verify Image') {
            steps {

                sh '''
                aws ecr describe-images \
                --repository-name ${ECR_REPO} \
                --region ${AWS_REGION}
                '''
            }
        }
    }

    post {

        success {

            echo '================================='
            echo 'Pipeline Completed Successfully'
            echo 'Image Pushed To Amazon ECR'
            echo '================================='
        }

        failure {

            echo '================================='
            echo 'Pipeline Failed'
            echo 'Check Console Output'
            echo '================================='
        }
    }
}