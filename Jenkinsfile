pipeline {
    agent { label 'docker-node' }
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t painting-app:v1 .'
            }
        }
        stage('Run') {
            steps {
                sh 'docker run -d -p 8082:80 painting-app:v1'
            }
        }
    }
}
