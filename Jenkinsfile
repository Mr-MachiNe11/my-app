pipeline {
    agent any

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp .'
            }
        }

        stage('Save Docker Image') {
            steps {
                sh 'docker save myapp | gzip > myapp.tar.gz'
            }
        }

        stage('Transfer Image to Production VM') {
            steps {
                sh 'scp myapp.tar.gz root@172.20.0.199:/root/'
            }
        }

        stage('Deploy on Production VM') {
            steps {
                sh '''
                ssh root@172.20.0.199 "
                    docker load < /root/myapp.tar.gz &&
                    docker rm -f myapp-container || true &&
                    docker run -d \
                    --name myapp-container \
                    -p 3001:3000 \
                    myapp
                "
                '''
            }
        }
    }
}
