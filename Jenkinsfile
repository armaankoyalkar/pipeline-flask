pipeline {
    agent any

    environment {
        IMAGE_NAME = "armaankoyalkar/flask-hospital-app"
        CONTAINER_NAME = "flask-hospital-app"
        TEST_CONTAINER = "test-container"
        PORT = "5000"
        TEST_PORT = "5001"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Test Container') {
            steps {
                sh """
                    docker rm -f ${TEST_CONTAINER} || true

                    docker run -d \
                        --name ${TEST_CONTAINER} \
                        -p ${TEST_PORT}:5000 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 10

                    echo "=== Logs ==="
                    docker logs ${TEST_CONTAINER} || true

                    echo "=== Health Check ==="
                    curl -f http://localhost:${TEST_PORT} || exit 1

                    docker rm -f ${TEST_CONTAINER} || true
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy Application') {
            steps {
                sh """
                    echo "Stopping old container if exists..."
                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Freeing port ${PORT} if occupied..."
                    docker ps -q --filter "publish=${PORT}" | xargs -r docker rm -f || true

                    echo "Starting new container..."
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${PORT}:5000 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        always {
            sh """
                docker rm -f ${TEST_CONTAINER} || true
                docker image prune -f || true
            """
        }

        success {
            echo "Pipeline Completed Successfully"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
