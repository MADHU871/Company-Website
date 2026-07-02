pipeline {
    agent any

    environment {
        IMAGE_NAME = "mad0008271/company-website"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                '''
            }
        }

        stage('Test Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    docker logout
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker info
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '''
                docker push %IMAGE_NAME%:%IMAGE_TAG%
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            bat 'docker logout'
        }

        failure {
            echo 'Pipeline failed!'
            bat 'docker logout'
        }
    }
}