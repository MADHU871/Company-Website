pipeline {

    agent any

    environment {

        IMAGE_NAME = "mad0008271/company-website"

    }

    stages {

        stage('Checkout') {

            steps {

                git 'https://github.com/MADHU871/Company-Website.git'

            }

        }

        stage('Build Docker Image') {

            steps {

                bat 'docker build -t %IMAGE_NAME% .'

            }

        }

        stage('Docker Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
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

        stage('Push Image') {

            steps {

                bat 'docker push %IMAGE_NAME%'

            }

        }

    }

}