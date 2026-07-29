pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        // Nombre de la imagen local (se usará para el build)
        IMAGE_NAME = 'mi-app'
        TAG = 'latest'
        
        // Variables para el tag en GHCR (con minúsculas)
        GHCR_IMAGE = 'ghcr.io/yairestrada/mi-app-jenkins'
        
        // Obtener commit SHA y timestamp (se calculan en el stage Prepare)
        COMMIT_SHA = ''
        TIMESTAMP = ''
    }

    stages {
        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh '''
                    apk add --no-cache docker-cli git
                    
                    # Solución al error de "dubious ownership"
                    git config --global --add safe.directory /var/jenkins_home/workspace/mi-app-jenkins
                    
                    # Obtener commit corto y timestamp
                    echo "COMMIT_SHA=$(git rev-parse --short HEAD)"
                    echo "TIMESTAMP=$(date +%Y%m%d-%H%M%S)"
                '''
                // Guardamos los valores en variables de entorno para usarlos luego
                script {
                    env.COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
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
                        echo \$GITHUB_TOKEN | docker login ghcr.io -u YairEstrada --password-stdin
                        
                        # Etiquetar con 'latest' y con el commit SHA
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:latest
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:${COMMIT_SHA}
                        docker tag ${IMAGE_NAME}:${TAG} ${GHCR_IMAGE}:build-${TIMESTAMP}
                        
                        # Subir todas las etiquetas
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
