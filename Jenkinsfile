pipeline {
    agent any

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                    aws s3 ls
                '''
            }
        }
    }
}
