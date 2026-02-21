pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('my-aws-iam-user')
        AWS_ACCESS_SECRET_KEY_ID = credentials('my-aws-iam-user')
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster1-Prod'
        AWS_ECS_TASK_DEF = 'LearnJenkinsApp-TaskDefinition1-Prod'
        AWS_ECS_SERVICE = 'LearnJenkinsApp-Service1-Prod'
    }

    stages {

        // stage('Build') {

        // }

        stage('Build Docker image') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            // steps {
            //     sh '''
            //         amazon-linux-extras install docker
            //         docker build -t myjenkinsapp .
            //     '''
            // }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-iam-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
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
