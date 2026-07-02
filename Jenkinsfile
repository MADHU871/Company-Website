pipeline {
    agent any

    triggers {
        // Poll every 5 minutes (optional)
        pollSCM('H/5 * * * *')

        // If using GitHub webhook, remove pollSCM
        // githubPush()
    }

    environment {
        IMAGE_NAME = "mad0008271/company-website"
        IMAGE_TAG  = "latest"
        CONTAINER  = "company-website-container"
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                bat """
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                """
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
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    """
                }
            }
        }

        stage('Tag Image') {
            steps {
                echo "Tagging image..."
                bat """
                docker tag %IMAGE_NAME%:%IMAGE_TAG% %IMAGE_NAME%:%BUILD_NUMBER%
                """
            }
        }

        stage('Push Latest Image') {
            steps {
                echo "Pushing latest image..."
                bat """
                docker push %IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Push Version Image') {
            steps {
                echo "Pushing version image..."
                bat """
                docker push %IMAGE_NAME%:%BUILD_NUMBER%
                """
            }
        }

        stage('Verify Push') {
            steps {
                echo "Pulling image from Docker Hub..."
                bat """
                docker pull %IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Stop Old Container') {
            steps {
                bat """
                docker stop %CONTAINER% || exit /b 0
                docker rm %CONTAINER% || exit /b 0
                """
            }
        }

        stage('Run New Container') {
            steps {
                bat """
                docker run -d ^
                --name %CONTAINER% ^
                -p 8080:80 ^
                %IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Container Status') {
            steps {
                bat "docker ps"
            }
        }

        stage('Container Logs') {
            steps {
                bat "docker logs %CONTAINER%"
            }
        }

        stage('Docker Images') {
            steps {
                bat "docker images"
            }
        }

        stage('Docker Logout') {
            steps {
                bat "docker logout"
            }
        }
    }

    post {

        success {
            echo "=================================="
            echo "Pipeline completed successfully."
            echo "Image pushed to Docker Hub."
            echo "Container deployed successfully."
            echo "=================================="
        }

        failure {
            echo "=================================="
            echo "Pipeline failed."
            echo "Check the failed stage in the Jenkins console."
            echo "=================================="
        }

        always {
            bat """
            docker logout || exit /b 0
            """
        }
    }
}