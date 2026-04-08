pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "197446684998"
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
                    -Dsonar.host.url=http://18.222.183.143:9000 \
                    -Dsonar.login=sqa_10732d9e1de88037d15dc58b312bba0e16da2412
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
        stage('Login & Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'TestUser'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    docker tag java-calculator:latest \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest

                    docker tag hi-app:latest \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest

                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest
                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest
                    '''
                }
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
                # 1. Create/Update the Deployment and the Service
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                
                # 2. Update to the latest ECR image
                kubectl set image deployment/java-calculator-deployment \
                java-calculator=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$CALC_REPO:latest

                kubectl rollout status deployment/java-calculator-deployment
                '''
            }
        }

        stage('Deploy HI App') {
            steps {
                sh '''
                # 1. Create/Update the Deployment and the Service
                kubectl apply -f hi-app/hi-deployment.yaml
                kubectl apply -f hi-app/hi-service.yaml
                kubectl apply -f hi-app/ingress.yaml

                # 2. Update to the latest ECR image
                kubectl set image deployment/hi-html-deployment \
                hi-html=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$HI_REPO:latest

                kubectl rollout status deployment/hi-html-deployment
                '''
            }
        }

            
    }
}
