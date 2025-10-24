pipeline {
    agent { label 'abhi-node' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        GITHUB_TOKEN = credentials('github')
        DOCKER_IMAGE = "abhi2310/paintingwebsite"
        SONAR_PROJECT_KEY = 'painting-website'
        SONARQUBE_TOKEN = credentials('SonarQube')
        SONAR_HOST_URL = 'http://localhost:9001'
        VERSION = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Start Pipeline') {
            steps {
                echo "Pipeline started successfully on node: ${env.NODE_NAME}"
                // Replaced cluster-level command with namespace-scoped check
                sh "kubectl get pods -n abhishekrajput2310-dev"
            }
        }

        stage('Checkout Code') {
            steps {
                echo "Starting Checkout Code stage"
                git branch: 'main', url: 'https://github.com/abhishek23102310/PaintingWebsite.git'
                echo "Checkout Code stage completed"
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            sonar-scanner \
                                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                -Dsonar.sources=. \
                                -Dsonar.projectVersion=${VERSION} \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.login=${SONARQUBE_TOKEN}
                        '''
                    }
                }
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
                sh "ls -la ~/.kube/"
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
