@library('sharedLib') _
pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_credentials'
        BRANCH_NAME = "${env.BRANCH_NAME}"
        IMAGE_TAG   = "${BRANCH_NAME}-${BUILD_NUMBER}"
        COMPOSE_FILE="docker-compose.yml"
    }
    stages {
        stage('Checkout Source') {
            steps {
                checkoutSource()
            }
        }
        stage('Docker Hub Login') {
            steps {
                dockerLogin('dockerhub_credentials')
            }
        }
        stage('Build & Tag Images') {
            steps {
                buildImages('./backend', './frontend', IMAGE_TAG)
            }
        }
        stage('Push Images to Docker Hub') {
            steps {
                pushImages()
            }
        }
        stage('Prepare .env for Compose') {
            steps {
                prepareEnvFile()
            }
        }
        stage('Deploy Environment') {
            steps {
                deployApp(COMPOSE_FILE, env.BRANCH_NAME)
            }
        }
        stage('Cleanup Local Images') {
            steps {
                cleanupImages()
            }
        }
    }
    post {
        success {
            echo "✅ ${BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${BRANCH_NAME}. Check logs."
        }
    }
}
