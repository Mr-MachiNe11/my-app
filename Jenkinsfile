pipeline {

    agent { label 'build-vm' }

    environment {

        IMAGE_NAME = "my-app"

        CONTAINER_NAME = "myapp-container"

        HOST_PORT = "3001"
        CONTAINER_PORT = "3000"

        TARGET_HOST = "172.20.0.199"
        TARGET_USER = "jenkins"

    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build \
                -t ${IMAGE_NAME}:${BUILD_NUMBER} .

                docker tag \
                ${IMAGE_NAME}:${BUILD_NUMBER} \
                ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Transfer Image') {
            steps {
                sh '''
                docker save \
                ${IMAGE_NAME}:${BUILD_NUMBER} \
                ${IMAGE_NAME}:latest | \
                ssh ${TARGET_USER}@${TARGET_HOST} "docker load"
                '''
            }
        }

        stage('Deploy') {

            agent {
                label 'prod-vm'
            }

            options {
                skipDefaultCheckout()
            }

            steps {

                sh '''
                docker rm -f ${CONTAINER_NAME} || true

                docker run -d \
                  --name ${CONTAINER_NAME} \
                  --restart always \
                  -p ${HOST_PORT}:${CONTAINER_PORT} \
                  ${IMAGE_NAME}:${BUILD_NUMBER}

                docker image prune -f
                '''

            }

        }

    }

    post {

        success {
            echo "Deployment successful: ${BUILD_NUMBER}"
        }

        failure {
            echo "Deployment failed"
        }

    }

}
