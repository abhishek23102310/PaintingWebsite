pipeline {
    agent { label 'Abhi-node' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dokcer-cred') // ID you set in Jenkins
        GITHUB_TOKEN = credentials('github-cred') // GitHub PAT
        DOCKER_IMAGE = "abhi2310/paintingwebsite"
        JIRA_ISSUE_KEY = "SCRUM-1"
        JIRA_SITE = "your-jira-site" // Set this in Jenkins Jira plugin
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
                        helm upgrade --install paintingwebsite ./helm-chart \
                        --set image.repository=${DOCKER_IMAGE} \
                        --set image.tag=${tag}
                    """
                }
            }
        }

        stage('Update Jira') {
            steps {
                jiraIssueSelector idOrKey: "${JIRA_ISSUE_KEY}"
                jiraAddComment comment: "Build #${env.BUILD_NUMBER} deployed successfully with Docker image tag ${env.BUILD_NUMBER}."
                jiraTransitionIssue transition: 'Done'
            }
        }
    }

    post {
        failure {
            jiraAddComment comment: "Build #${env.BUILD_NUMBER} failed. Please check Jenkins logs."
        }
    }
}
