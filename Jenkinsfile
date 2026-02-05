pipeline {
    agent any

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }
        }
    }
}
