pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = "ghcr.io/yairestrada/mi-app-jenkins"
    }

    stages {

        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh 'apk add --no-cache docker-cli git'

                script {
                    def COMMIT_SHA = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def BUILD_TIMESTAMP = sh(script: "date '+%Y%m%d-%H%M%S'", returnStdout: true).trim()
                    env.IMAGE_TAG = "${COMMIT_SHA}-${BUILD_TIMESTAMP}"
                }

                echo "Tag: ${env.IMAGE_TAG}"
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
                sh """
                docker build -t ${IMAGE_NAME}:latest .
                docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${env.IMAGE_TAG}
                """
            }
        }

        stage('Push') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                    echo \$GITHUB_TOKEN | docker login ghcr.io -u yairestrada --password-stdin
                    docker push ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:${env.IMAGE_TAG}
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
        always {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f || true'
        }
    }
}     
