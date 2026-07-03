pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        GIT_URL         = 'https://github.com/MADHU871/Company-Website.git'
        GIT_BRANCH      = 'main'

        DOCKER_IMAGE    = 'mad0008271/company-website'
        CONTAINER_NAME  = 'company-website'

        HOST_PORT       = '8081'
        CONTAINER_PORT  = '80'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo 'Checking out source code...'

                git branch: "${GIT_BRANCH}",
                    url: "${GIT_URL}"
                    // credentialsId: 'YOUR_GITHUB_TOKEN_CREDENTIAL_ID'
            }
        }

        stage('Verify Docker') {
            steps {
                bat 'docker --version'
                bat 'docker info'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t %DOCKER_IMAGE%:latest .
                '''
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

        stage('Tag Image') {
            steps {

                bat '''
                docker tag %DOCKER_IMAGE%:latest %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''

            }
        }

        stage('Push Latest Image') {
            steps {

                bat '''
                docker push %DOCKER_IMAGE%:latest
                '''

            }
        }

        stage('Push Build Number Image') {
            steps {

                bat '''
                docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''

            }
        }

        stage('Verify Push') {
            steps {

                bat '''
                docker pull %DOCKER_IMAGE%:latest
                '''

            }
        }

        stage('Stop Old Container') {
            steps {

                bat '''
                docker stop %CONTAINER_NAME% || exit /b 0
                '''

            }
        }

        stage('Remove Old Container') {
            steps {

                bat '''
                docker rm %CONTAINER_NAME% || exit /b 0
                '''

            }
        }

        stage('Run New Container') {
            steps {

                bat '''
                docker run -d ^
                --name %CONTAINER_NAME% ^
                -p %HOST_PORT%:%CONTAINER_PORT% ^
                %DOCKER_IMAGE%:latest
                '''

            }
        }

        stage('Container Status') {
            steps {

                bat '''
                docker ps -a
                '''

            }
        }

        stage('Container Logs') {
            steps {

                bat '''
                docker logs %CONTAINER_NAME%
                '''

            }
        }

        stage('Docker Images') {
            steps {

                bat '''
                docker images
                '''

            }
        }

        stage('Docker Logout') {
            steps {

                bat '''
                docker logout
                '''

            }
        }
    }

    post {

        success {

            echo "==========================================="
            echo "BUILD SUCCESS"
            echo "Docker Image Built Successfully"
            echo "Docker Image Pushed Successfully"
            echo "Container Running Successfully"
            echo "Website URL : http://localhost:8081"
            echo "==========================================="
        }

        failure {

            bat 'docker logout || exit /b 0'

            echo "==========================================="
            echo "BUILD FAILED"
            echo "Check Jenkins Console Output"
            echo "==========================================="
        }

        always {

            cleanWs()
        }
    }
}