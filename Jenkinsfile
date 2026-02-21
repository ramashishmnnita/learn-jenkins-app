pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_ACCESS_KEY_ID = credentials('my-aws-iam-user')
        AWS_ACCESS_SECRET_KEY_ID = credentials('my-aws-iam-user')
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_DOCKER_REGISTRY = '081338828869.dkr.ecr.ap-south-1.amazonaws.com'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster1-Prod'
        AWS_ECS_TASK_DEF = 'LearnJenkinsApp-TaskDefinition1-Prod'
        AWS_ECS_SERVICE = 'LearnJenkinsApp-Service1-Prod'
    }

    stages {
        stage('Build Base Image') {
            steps {
                // we can use seperate pipeline for this but lets keep as it is now
                sh 'docker build -f ci/Dockerfile-aws-cli -t my-aws-cli .'
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:25-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Hello"
            }
        }

        stage('Build Docker image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-iam-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-iam-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEF:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }
    }
}
