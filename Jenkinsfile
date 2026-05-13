pipeline {
    agent any

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Mr-MachiNe11/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker rm -f myapp-container || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                docker run -d \
                --name myapp-container \
                -p 3000:3000 \
                myapp
                '''
            }
        }
    }
}
