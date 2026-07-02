pipeline {
    agent any

    environment {
        IMAGE_NAME = "mad0008271/company-website"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push %IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Verify Image') {
            steps {
                bat "docker images"
            }
        }

        stage('Docker Logout') {
            steps {
                bat "docker logout"
            }
        }

        stage('Cleanup Local Image') {
            steps {
                bat "docker image rm %IMAGE_NAME%:%IMAGE_TAG% || exit /b 0"
            }
        }
    }

    post {
        success {
            echo '✅ Docker image built and pushed successfully!'
        }

        failure {
            echo '❌ Pipeline failed!'
        }

        always {
            bat 'docker logout || exit /b 0'
        }
    }
}