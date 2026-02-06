pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('my-aws-iam-user')
        AWS_ACCESS_SECRET_KEY_ID = credentials('my-aws-iam-user')
    }

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-iam-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
                    '''
                }
            }
        }
    }
}
