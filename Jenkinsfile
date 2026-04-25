pipeline {
    agent any

    options {
        timeout(time: 2, unit: 'MINUTES')
    }

    environment {
        DOCKER_API_VERSION = "1.42"
        DOCKER_HOST = "unix:///var/run/docker.sock"
        
        NEXUS_URL = "http://nexus:8083"
        NEXUS_HOST = "nexus:8083"
        
        CREDENTIALS_ID = "nexus-creds"          
        IMAGE_NAME = "sumador"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker version || true
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Run tests') {
            steps {
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} npm test"
            }
        }
        
        stage('Tag Docker Image') {
            steps {
                echo "Tagging Docker image for Nexus repository..."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${NEXUS_HOST}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: CREDENTIALS_ID,
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                        echo \${NEXUS_PASS} | docker login ${NEXUS_HOST} -u \${NEXUS_USER} --password-stdin
                        docker push ${NEXUS_HOST}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ${NEXUS_HOST}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images..."
            sh """
                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                docker rmi ${NEXUS_HOST}/${IMAGE_NAME}:${IMAGE_TAG} || true
            """
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}