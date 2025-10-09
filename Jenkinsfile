pipeline {
    agent { label 'abhi-node' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred') // DockerHub credentials ID
        GITHUB_TOKEN = credentials('github-cred') // GitHub PAT
        DOCKER_IMAGE = "abhi2310/paintingwebsite"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/abhishek23102310/PaintingWebsite.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}:${tag}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh """
                        helm upgrade --install paintingwebsite ./paintingwebsite-chart \
                        --set image.repository=${DOCKER_IMAGE} \
                        --set image.tag=${tag}
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Build #${env.BUILD_NUMBER} failed. Please check Jenkins logs."
        }
    }
}
