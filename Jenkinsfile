pipeline {
    agent any

    environment {
        IMAGE_NAME = "ghcr.io/yairestrada/mi-app-jenkins"
    }

    stages {

        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh 'apk add --no-cache docker-cli git || true'

                script {
                    COMMIT_SHA = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    BUILD_TIMESTAMP = sh(script: "date '+%Y%m%d-%H%M%S'", returnStdout: true).trim()
                    IMAGE_TAG = "${COMMIT_SHA}-${BUILD_TIMESTAMP}"
                }

                echo "Tag: ${IMAGE_TAG}"
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:latest .
                    docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                    echo \$GITHUB_TOKEN | docker login ghcr.io -u yairestrada --password-stdin
                    docker push ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Verify') {
            steps {
                echo '✅ Imagen subida correctamente a GHCR'
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline falló!'
        }
        always {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f'
        }
    }
}
