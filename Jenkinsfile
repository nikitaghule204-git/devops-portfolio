pipeline {
    agent any
environment {
    IMAGE_NAME   = "devops-portfolio"
    AWS_REGION   = "ap-south-1"
    ECR_HOST     = "759729568917.dkr.ecr.ap-south-1.amazonaws.com"
    ECR_REGISTRY = "759729568917.dkr.ecr.ap-south-1.amazonaws.com/devops-portfolio"
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
                sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=devops-portfolio -Dsonar.projectName=devops-portfolio -Dsonar.sources=."
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
        withCredentials([
            usernamePassword(
                credentialsId: 'aws-creds',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_HOST}

                docker push ${ECR_REGISTRY}:${BUILD_NUMBER}
            '''
        }
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
