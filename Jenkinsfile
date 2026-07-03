pipeline {
    agent any

    environment {
        GITHUB_REPO   = 'https://github.com/MADHU871/Company-Website.git'
        DOCKER_IMAGE  = 'mad0008271/company-website'
        CONTAINER     = 'company-website'
        PORT          = '8081'
    }

    triggers {
        pollSCM('H/2 * * * *')    // Check GitHub every 2 minutes
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo 'Checking out source code...'
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: "${GITHUB_REPO}"
            }
        }

        stage('Docker Version') {
            steps {
                bat 'docker --version'
                bat 'docker info'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
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
                bat 'docker tag %DOCKER_IMAGE%:latest %DOCKER_IMAGE%:%BUILD_NUMBER%'
            }
        }

        stage('Push Latest Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%:latest'
            }
        }

        stage('Push Build Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%:%BUILD_NUMBER%'
            }
        }

        stage('Verify Docker Images') {
            steps {
                bat 'docker images'
            }
        }

        stage('Stop Old Container') {
            steps {
                bat 'docker stop %CONTAINER% || exit /b 0'
            }
        }

        stage('Remove Old Container') {
            steps {
                bat 'docker rm %CONTAINER% || exit /b 0'
            }
        }

        stage('Run New Container') {
            steps {
                bat 'docker run -d --name %CONTAINER% -p %PORT%:80 %DOCKER_IMAGE%:latest'
            }
        }

        stage('Container Status') {
            steps {
                bat 'docker ps -a'
            }
        }

        stage('Container Logs') {
            steps {
                bat 'docker logs %CONTAINER%'
            }
        }

        stage('Docker Logout') {
            steps {
                bat 'docker logout'
            }
        }
    }

    post {

        success {
            echo '========================================='
            echo 'BUILD SUCCESSFUL'
            echo 'Docker image built and pushed'
            echo 'Container deployed successfully'
            echo '========================================='
        }

        failure {
            bat 'docker logout || exit /b 0'
            echo '========================================='
            echo 'BUILD FAILED'
            echo 'Check Jenkins Console Output'
            echo '========================================='
        }

        always {
            cleanWs()
        }
    }
}