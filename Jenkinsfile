pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build docker image') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.17.9'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps {
                sh '''
                    amazon-linux-extras install docker
                    docker build -t my-jenkinsapp .'
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.17.9'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId:'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY',usernameVariable:'AWS_ACCESS_KEY_ID')]{
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster learnjenkins-cluster --service learnjenkins-service --task-definition learnjenkins-definition:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster learnjenkins-cluster --service learnjenkins-service
                    '''
                })
            }
        }
    }
}
