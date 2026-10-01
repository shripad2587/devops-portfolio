pipeline {
    agent any

    environment {
        IMAGE_NAME   = "devops-portfolio"
        AWS_REGION   = "us-east-1"
        ECR_REGISTRY = "public.ecr.aws/j0h7e6b5/devops-portfolio"
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
                        sh "${scannerHome}/bin/sonar-scanner"
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
                        aws ecr-public get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin public.ecr.aws
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag \
                    ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push To ECR') {
            steps {
                sh '''
                    docker push ${ECR_REGISTRY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-creds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws ecr-public describe-images \
                        --repository-name ${IMAGE_NAME} \
                        --region ${AWS_REGION}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Image Successfully Uploaded To ECR Public'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
