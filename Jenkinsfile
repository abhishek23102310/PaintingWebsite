pipeline {
    agent { label 'docker-node' }

    triggers {
        pollSCM('* * * * *') // or use GitHub webhook
    }

    stages {
        stage('Build') {
            steps {
                echo "Building app from ${env.BRANCH_NAME}"
                sh 'docker build -t jira-app:v1 .'
            }
        }
        stage('Deploy') {
            when {
                branch 'feature/AD-5'
            }
            steps {
                echo "Deploying app from Jira branch"
                sh 'docker run -d -p 8082:80 jira-app:v1'
            }
        }
    }

    post {
        success {
            echo "Build succeeded"
            // Add Jira integration here
        }
        failure {
            echo "Build failed"
        }
    }
}
