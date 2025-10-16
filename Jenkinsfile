pipeline {
    agent { label 'abhi-node' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred') // DockerHub credentials ID
        GITHUB_TOKEN = credentials('github') // GitHub PAT
        DOCKER_IMAGE = "abhi2310/paintingwebsite"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Starting Checkout Code stage"
                git branch: 'main', url: 'https://github.com/abhishek23102310/PaintingWebsite.git'
                echo "Checkout Code stage completed"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Starting Build Docker Image stage"
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
                }
                echo "Build Docker Image stage completed"
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Starting Push Docker Image stage"
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}:${tag}
                    """
                }
                echo "Push Docker Image stage completed"
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                echo "Starting Deploy to Kubernetes with Helm stage"
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh """
                        helm upgrade --install paintingwebsite ./paintingwebsite-chart \
                        --set image.repository=${DOCKER_IMAGE} \
                        --set image.tag=${tag}
                    """
                }
                echo "Deploy to Kubernetes with Helm stage completed"
            }
        }
    }

    post {
        failure {
            echo "Build #${env.BUILD_NUMBER} failed. Please check Jenkins logs."
        }
    }
}
