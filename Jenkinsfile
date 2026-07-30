@Library('sharedLib') _
pipeline {
    agent any

    environment {
        // Branch name: use GIT_BRANCH (set by checkout), fallback to 'main'
        BRANCH_NAME = "${env.GIT_BRANCH ?: 'main'}"
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
                dockerLogin('dockerhub-credentials')
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
                    // Manual approval only for staging/production
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
