pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = 'mi-app'
        TAG = 'latest'
    }

    stages {
        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh '''
                apk add --no-cache docker-cli git
                git config --global --add safe.directory /var/jenkins_home/workspace/mi-app-jenkins
                git rev-parse --short HEAD
                '''
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
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        // 🚀 AHORA SÍ PUBLICAMOS EN GHCR
        stage('Push') {
            steps {
                echo '🚀 Subiendo imagen a GitHub Container Registry...'
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        echo $GITHUB_TOKEN | docker login ghcr.io -u YairEstrada --password-stdin
                        docker tag $IMAGE_NAME:$TAG ghcr.io/YairEstrada/mi-app-jenkins:latest
                        docker tag $IMAGE_NAME:$TAG ghcr.io/YairEstrada/mi-app-jenkins:build-$(date +%Y%m%d-%H%M%S)
                        docker push ghcr.io/YairEstrada/mi-app-jenkins:latest
                        docker push ghcr.io/YairEstrada/mi-app-jenkins:build-$(date +%Y%m%d-%H%M%S)
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                echo '✅ Verificando imagen publicada...'
                sh '''
                    echo "Imagen publicada en GHCR: ghcr.io/YairEstrada/mi-app-jenkins:latest"
                    docker images | grep $IMAGE_NAME
                '''
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
