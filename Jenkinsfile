pipeline{
    agent any
    environment{
        AWS_DOCKER_REGISTRY = '871749382674.dkr.ecr.us-east-2.amazonaws.com'
        APP_NAME = 'react-cicd'
        AWS_DEFAULT_REGION = 'us-east-2'
    }
    stages{
        stage('Build'){
            agent{
                docker{
                    image 'node:24.14.0-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test'){
            agent{
                docker{
                    image 'node:24.14.0-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('Build My Image'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'reactAWS', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {

                    sh '''
                        dnf install -y docker
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME .
                        docker images

                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
                    '''
                }
            }
        }
        stage('Deploy to AWS ECS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'reactAWS', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {
                    sh '''
                        aws --version

                        yum install jq -y

                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster my-react-cluster-20260330 --service my-react-service-20260330 --task-definition my-react-task-definition-json-20260330:$LATEST_TD_REVISION
                    '''
                }
            }
        }

    }
}