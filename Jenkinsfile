pipeline {
    agent any

    environment {
        APP_NAME = 'my-app'
        IMAGE_NAME = 'rakib063/my-app:latest'

        PROD_SERVER = '172.20.0.199'
        PROD_USER = 'root'

        CONTAINER_NAME = 'myapp-container'
        HOST_PORT = '3001'
        CONTAINER_PORT = '3000'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
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

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push ${IMAGE_NAME}'
            }
        }

        stage('Deploy to Production VM') {
            steps {
                sh """
                ssh ${PROD_USER}@${PROD_SERVER} '
                    docker pull ${IMAGE_NAME} &&
                    docker rm -f ${CONTAINER_NAME} || true &&
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p ${HOST_PORT}:${CONTAINER_PORT} \
                      ${IMAGE_NAME}
                '
                """
            }
        }
    }
}
