pipeline {
    agent any

    environment {
        DOCKER_REPO = 'rakib063/my-app'

        PROD_SERVER = '172.20.0.199'
        PROD_USER = 'root'

        CONTAINER_NAME = 'myapp-container'
        HOST_PORT = '3001'
        CONTAINER_PORT = '3000'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${DOCKER_REPO}:${BUILD_NUMBER} .
                docker tag ${DOCKER_REPO}:${BUILD_NUMBER} ${DOCKER_REPO}:latest
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
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

        stage('Push Images to Docker Hub') {
            steps {
                sh '''
                docker push ${DOCKER_REPO}:${BUILD_NUMBER}
                docker push ${DOCKER_REPO}:latest
                '''
            }
        }

        stage('Deploy to Production VM') {
            steps {
                sh """
                ssh ${PROD_USER}@${PROD_SERVER} '
                    docker pull ${DOCKER_REPO}:${BUILD_NUMBER} &&
                    docker rm -f ${CONTAINER_NAME} || true &&
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_REPO}:${BUILD_NUMBER} &&
                    docker image prune -f
                '
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful. Version: ${BUILD_NUMBER}"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
