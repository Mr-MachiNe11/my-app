pipeline {
    agent any

    environment {
        APP_NAME = 'myapp'
        IMAGE_FILE = 'myapp.tar.gz'
        PROD_SERVER = '172.20.0.199'
        PROD_USER = 'root'
        CONTAINER_NAME = 'myapp-container'
        HOST_PORT = '3001'
        CONTAINER_PORT = '3000'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${APP_NAME} .'
            }
        }

        stage('Save Docker Image') {
            steps {
                sh 'docker save ${APP_NAME} | gzip > ${IMAGE_FILE}'
            }
        }

        stage('Transfer Image to Production VM') {
            steps {
                sh 'scp ${IMAGE_FILE} ${PROD_USER}@${PROD_SERVER}:/root/'
            }
        }

        stage('Deploy on Production VM') {
            steps {
                sh '''
                ssh ${PROD_USER}@${PROD_SERVER} "
                    docker load < /root/${IMAGE_FILE} &&
                    docker rm -f ${CONTAINER_NAME} || true &&
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p ${HOST_PORT}:${CONTAINER_PORT} \
                    ${APP_NAME}
                "
                '''
            }
        }
    }
}
