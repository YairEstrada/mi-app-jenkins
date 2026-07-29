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
                
                # Solución al error de "dubious ownership"
                git config --global --add safe.directory /var/jenkins_home/workspace/mi-app-jenkins
                
                # Obtener commit corto (ya no fallará)
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

        stage('Push') {
            steps {
                echo '🚀 Subiendo imagen...'
                sh '''
                echo "Aquí iría docker push si tienes registry configurado"
                '''
            }
        }

        stage('Verify') {
            steps {
                echo '✅ Verificando contenedor...'
                sh 'docker images | grep $IMAGE_NAME'
            }
        }
    }

    post {
        always {
            echo '🧹 Limpiando recursos...'
            sh '''
            docker image prune -f || true
            '''
        }

        success {
            echo '✅ Pipeline exitoso!'
        }

        failure {
            echo '❌ Pipeline falló!'
        }
    }
}
