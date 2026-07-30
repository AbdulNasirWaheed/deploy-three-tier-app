@Library('sharedLib') _
pipeline {
    agent any

    environment {
        // Get branch name – fallback to 'main' if the command returns nothing
        BRANCH_NAME = sh(
            returnStdout: true,
            script: '''
                git branch -r --contains HEAD | grep -o "origin/.*" | sed "s|origin/||" | head -1 || echo main
            '''
        ).trim()

        IMAGE_TAG    = "${BRANCH_NAME}-${env.BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.${BRANCH_NAME}.yml"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Docker Login') {
            steps {
                dockerLogin('dockerhub_credentials')
            }
        }
        stage('Build Images') {
            steps {
                buildImages('./backend', './frontend', IMAGE_TAG)
            }
        }
        stage('Push Images') {
            steps {
                pushImages()
            }
        }
        stage('Prepare .env') {
            steps {
                prepareEnvFile()
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (BRANCH_NAME == 'stg' || BRANCH_NAME == 'prod') {
                        input message: "Deploy to ${BRANCH_NAME}?", ok: "Deploy"
                    }
                    deployApp(COMPOSE_FILE, BRANCH_NAME)
                }
            }
        }
        stage('Cleanup') {
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
