pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "059941825967"
        AWS_REGION = "ap-south-1"
        CALC_REPO = "java-calculator-repo"
        HI_REPO = "welcome"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/devopswithanisha/kubernetes-java-app.git'
            }
        }

        stage('Build Java Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Calculator Docker Image') {
            steps {
                sh '''
                docker build -t java-calculator .
                '''
            }
        }

        stage('Build HI App Docker Image') {
            steps {
                sh '''
                docker build -t hi-app ./hi-app
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION |
                docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Calculator Image') {
            steps {
                sh '''
                docker tag java-calculator:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest
                '''
            }
        }

        stage('Push HI App Image') {
            steps {
                sh '''
                docker tag hi-app:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest
                '''
            }
        }

        stage('Deploy Calculator App') {
            steps {
                sh '''
                kubectl set image deployment/java-calculator-deployment \
                calculator=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest
                '''
            }
        }

        stage('Deploy HI App') {
            steps {
                sh '''
                kubectl set image deployment/hi-deployment \
                hi-container=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest
                '''
            }
        }

    }
}
