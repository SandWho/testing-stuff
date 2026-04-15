pipeline {
    agent any

    stages {

        stage('Test') {
            agent {
                docker {
                   image 'node:20-alpine'
                   reuseNode true
                 }
            }
            steps {
                dir('react-cicd'){ // Change to the react-cicd directory
                  sh '''
                  npm install
                  npm test
                 '''
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                 }
            }
            steps {
                dir('react-cicd') {
                  sh '''
                    npm install
                    npm run build
                  '''
               }
            }
        }

        stage('Build My Docker Image'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my_cred1', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
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
    }

    post {
        success {
            echo 'Build and tests succeeded!'
        }
        failure {
            echo 'Build or tests failed.'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}