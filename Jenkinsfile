pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
        }
    }

    environment {
        GITHUB_USER = 'YairEstrada'
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'YairEstrada/mi-app-jenkins'
        COMMIT_SHA = ''
        BUILD_TIMESTAMP = ''
        IMAGE_TAG_LATEST = ''
        IMAGE_TAG_COMMIT = ''
        IMAGE_TAG_BUILD = ''
    }

    stages {
        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh 'apk add --no-cache docker-cli git'
                sh 'docker --version'
                sh 'node --version'
                sh 'npm --version'
                sh "git config --global --add safe.directory ${env.WORKSPACE}"
                sh "echo 'Workspace: ${env.WORKSPACE}'"

                script {
                    env.COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
                    env.IMAGE_TAG_LATEST = "${env.REGISTRY}/${env.IMAGE_NAME}:latest"
                    env.IMAGE_TAG_COMMIT = "${env.REGISTRY}/${env.IMAGE_NAME}:${env.COMMIT_SHA}"
                    env.IMAGE_TAG_BUILD = "${env.REGISTRY}/${env.IMAGE_NAME}:build-${env.BUILD_TIMESTAMP}"
                    echo "COMMIT_SHA=${env.COMMIT_SHA}"
                    echo "BUILD_TIMESTAMP=${env.BUILD_TIMESTAMP}"
                    echo "IMAGE_TAG_COMMIT=${env.IMAGE_TAG_COMMIT}"
                }
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
                    docker.build("${env.IMAGE_TAG_COMMIT}")
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        echo \$GITHUB_TOKEN | docker login ghcr.io -u ${env.GITHUB_USER} --password-stdin
                        docker tag ${env.IMAGE_TAG_COMMIT} ${env.IMAGE_TAG_LATEST}
                        docker tag ${env.IMAGE_TAG_COMMIT} ${env.IMAGE_TAG_BUILD}
                        docker push ${env.IMAGE_TAG_COMMIT}
                        docker push ${env.IMAGE_TAG_LATEST}
                        docker push ${env.IMAGE_TAG_BUILD}
                    """
                }
            }
        }

        stage('Verify') {
            steps {
                sh """
                    echo "✅ Imagen publicada: ${env.IMAGE_TAG_COMMIT}"
                    echo "Tags: latest, build-${env.BUILD_TIMESTAMP}"
                """
            }
        }
    }

    post {
        success { echo '🎉 Pipeline completado exitosamente!' }
        failure { echo '❌ Pipeline falló!' }
        cleanup {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f || true'
        }
    }
}
