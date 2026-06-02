pipeline {
agent any

```
environment {
    IMAGE_NAME = "armaankoyalkar/flask-hospital-app"
    DOCKERHUB_CREDENTIALS = "dockerhub-creds"
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
                docker rm -f test-container || true

                docker run -d \
                  --name test-container \
                  -p 5001:5000 \
                  ${IMAGE_NAME}:${BUILD_NUMBER}

                sleep 10

                curl -f http://localhost:5001

                docker rm -f test-container
            """
        }
    }

    stage('Docker Login') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                '''
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
                docker rm -f flask-hospital-app || true

                docker run -d \
                  --name flask-hospital-app \
                  -p 5000:5000 \
                  ${IMAGE_NAME}:${BUILD_NUMBER}
            """
        }
    }
}

post {
    always {
        sh '''
            docker rm -f test-container || true
            docker image prune -f
        '''
    }

    success {
        echo 'Pipeline Completed Successfully'
    }

    failure {
        echo 'Pipeline Failed'
    }
}
```

}
