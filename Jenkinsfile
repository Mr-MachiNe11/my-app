pipeline {
    agent none

    environment {
        IMAGE_NAME = "myapp"
        CONTAINER_NAME = "myapp-container"
        PROD_SERVER = "root@172.20.0.199"
        HOST_PORT = "3001"
        CONTAINER_PORT = "3000"
    }

    stages {

        stage('Build Docker Image') {
            agent { label 'build-vm' }

            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
                sh 'docker save ${IMAGE_NAME} | gzip > ${IMAGE_NAME}.tar.gz'
            }
        }

        stage('Transfer Image to Production VM') {
            agent { label 'build-vm' }

            steps {
                sh 'scp ${IMAGE_NAME}.tar.gz ${PROD_SERVER}:/root/'
            }
        }

        stage('Deploy on Production VM') {
            agent { label 'prod-vm' }

            steps {
                sh """
                docker load < /root/${IMAGE_NAME}.tar.gz &&
                docker rm -f ${CONTAINER_NAME} || true &&
                docker run -d \
                --name ${CONTAINER_NAME} \
                -p ${HOST_PORT}:${CONTAINER_PORT} \
                ${IMAGE_NAME}
                """
            }
        }
    }
}
