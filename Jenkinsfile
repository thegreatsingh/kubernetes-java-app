pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "858321320845"
        AWS_REGION = "us-east-2"
        CALC_REPO = "java-calculator-repo"
        HI_REPO = "welcome"
        IMAGE_TAG = "latest"
    }
    stages {

        stage('Build Java Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=java-calculator \
                    -Dsonar.host.url=http://18.118.1.22 \
                    -Dsonar.login=sqa_9170b81b4c471396a60a3484d6ec1cab452d8302
                    '''
                }
            }
        }

        stage('Build Calculator Docker Image') {
            steps {
                sh 'docker build -t java-calculator .'
            }
        }

        stage('Build HI App Docker Image') {
            steps {
                sh 'docker build -t hi-app ./hi-app'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Calculator Image') {
            steps {
                sh '''
                docker tag java-calculator:latest \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest

                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest
                '''
            }
        }

        stage('Push HI App Image') {
            steps {
                sh '''
                docker tag hi-app:latest \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest

                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest
                '''
            }
        }

        stage('Deploy Calculator App') {
            steps {
                sh '''
                kubectl set image deployment/java-calculator-deployment \
                java-calculator=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest

                kubectl rollout restart deployment/java-calculator-deployment
                kubectl rollout status deployment/java-calculator-deployment
                '''
            }
        }

        stage('Deploy HI App') {
            steps {
                sh '''
                kubectl set image deployment/hi-html-deployment \
                hi-html=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest

                kubectl rollout restart deployment/hi-html-deployment
                kubectl rollout status deployment/hi-html-deployment
                '''
            }
        }
    }
}
