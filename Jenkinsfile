pipeline {
    agent none

    environment {
        IMAGE_NAME = "myapp"
        CONTAINER_NAME = "myapp-container"
        PROD_SERVER = "root@172.20.0.199"
        HOST_PORT = "3001"
        CONTAINER_PORT = "3000"
        TAR_FILE = "myapp.tar.gz"
    }

    stages {

        stage('Build Docker Image') {
            agent { label 'build-vm' }

            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
                sh 'docker save ${IMAGE_NAME} | gzip > ${TAR_FILE}'
            }
        }

        stage('Transfer Image to Production VM') {
            agent { label 'build-vm' }

            steps {
                sh 'scp ${TAR_FILE} ${PROD_SERVER}:/tmp/'
            }
        }

        stage('Deploy on Production VM') {
            agent { label 'prod-vm' }

            options {
                skipDefaultCheckout()
            }

            steps {
                sh """
                ssh ${PROD_SERVER} '
                    docker load < /tmp/${TAR_FILE} &&
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
