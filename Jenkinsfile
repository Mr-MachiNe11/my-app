pipeline {

    agent { label 'build-vm' }

    environment {

        IMAGE_NAME = "my-app"

        CONTAINER_NAME = "myapp-container"

        HOST_PORT = "3001"
        CONTAINER_PORT = "3000"

        TARGET_HOST = "172.20.0.199"
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
                    ${IMAGE_NAME}:latest \
                    | gzip \
                    > ${WORKSPACE}/my-app.tar.gz
                '''

                stash name: 'docker-image', includes: 'my-app.tar.gz'
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

                unstash 'docker-image'

                sh '''
                    echo "Running deployment on:"
                    hostname

                    echo "Loading Docker image..."
                    gunzip -c my-app.tar.gz | docker load

                    echo "Removing old container..."
                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Starting container..."
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart always \
                      -p ${HOST_PORT}:${CONTAINER_PORT} \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "Deployment result:"
                    docker ps
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