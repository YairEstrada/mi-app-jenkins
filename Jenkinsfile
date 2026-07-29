pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        // Variables de entorno que usaremos en todo el pipeline
        GITHUB_USER = 'YairEstrada'
        IMAGE_NAME = 'mi-app'
        TAG = 'latest'
        GHCR_IMAGE = 'ghcr.io/yairestrada/mi-app-jenkins'
        // Declaramos las variables que se asignarán en Prepare
        COMMIT_SHA = ''
        TIMESTAMP = ''
    }

    stages {
        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh '''
                    apk add --no-cache docker-cli git
                    git config --global --add safe.directory /var/jenkins_home/workspace/mi-app-jenkins
                '''
                script {
                    // Asignamos los valores a las variables de entorno
                    env.COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
                    echo "COMMIT_SHA=${env.COMMIT_SHA}"
                    echo "TIMESTAMP=${env.TIMESTAMP}"
                }
            }
        }

        stage('Install') {
            steps {
                echo '📦 Instalando dependencias...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Ejecutando tests...'
                sh 'npm test || true'
            }
        }

        stage('Build') {
            steps {
                echo '🏗️ Construyendo imagen Docker...'
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push') {
            steps {
                echo '🚀 Subiendo imagen a GitHub Container Registry...'
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        echo \$GITHUB_TOKEN | docker login ghcr.io -u ${GITHUB_USER} --password-stdin
                        
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:latest
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:${COMMIT_SHA}
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:build-${TIMESTAMP}
                        
                        docker push ${GHCR_IMAGE}:latest
                        docker push ${GHCR_IMAGE}:${COMMIT_SHA}
                        docker push ${GHCR_IMAGE}:build-${TIMESTAMP}
                    """
                }
            }
        }

        stage('Verify') {
            steps {
                echo '✅ Verificando imagen publicada...'
                sh """
                    echo "📦 Imagen publicada: ${GHCR_IMAGE}"
                    echo "Tags: latest, ${COMMIT_SHA}, build-${TIMESTAMP}"
                    docker images | grep ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        always {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f || true'
        }
        success {
            echo '✅ Pipeline exitoso!'
        }
        failure {
            echo '❌ Pipeline falló!'
        }
    }
}
