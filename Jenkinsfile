pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('my-aws-iam-user')
        AWS_ACCESS_SECRET_KEY_ID = credentials('my-aws-iam-user')
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {
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
                        LATEST_TD_REVISION = $(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster LearnJenkinsApp-Cluster1-Prod --service LearnJenkinsApp-Service1-Prod --task-definition LearnJenkinsApp-TaskDefinition1-Prod:6
                    '''
                }
            }
        }
    }
}
