pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-bluegreen-app"
        BLUE_PORT = "8088"
        GREEN_PORT = "8087"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Deploy to Green Environment') {
            steps {
                script {
                    bat """
                        docker stop green  echo Green container not running
                        docker rm green  echo Green container not found
                        docker run -d --name green -p ${GREEN_PORT}:80 ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    def blueContainer = bat(script: 'docker ps -q -f name=blue', returnStdout: true).trim()
                    if (blueContainer) {
                        echo "Stopping and removing old blue container..."
                        def stopStatus = bat(script: 'docker stop blue', returnStatus: true)
                        def rmStatus = bat(script: 'docker rm blue', returnStatus: true)
                        if (stopStatus != 0 || rmStatus != 0) {
                            echo "There was an issue stopping/removing blue, but continuing..."
                        }
                    } else {
                        echo "No existing blue container to stop/remove"
                    }

                    bat "docker rename green blue"
                }
            }
        }
    }
}