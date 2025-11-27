def dockerImageServer
def dockerImageClient

pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'doc'
        IMAGE_NAME_SERVER = 'yasminekriaa21/mern-server'
        IMAGE_NAME_CLIENT = 'yasminekriaa21/mern-client'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}:latest")
                    }
                }
            }
        }

        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}:latest")
                    }
                }
            }
        }

        stage('Scan Server Image') {
            steps {
                script {
                    bat """
                    docker run --rm ^
                    -v //./pipe/docker_engine://./pipe/docker_engine ^
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_SERVER}:latest
                    """
                }
            }
        }

        stage('Scan Client Image') {
            steps {
                script {
                    bat """
                    docker run --rm ^
                    -v //./pipe/docker_engine://./pipe/docker_engine ^
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_CLIENT}:latest
                    """
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS_ID) {
                        dockerImageServer.push('latest')
                        dockerImageClient.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            bat "docker system prune -af"  
        }
    }
}
