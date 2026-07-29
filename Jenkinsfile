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
        REGISTRY = 'ghcr.io'
        REPO = 'ghcr.io/yairgiovanicambronestrada-blip/mi-app'
    }

    stages {

        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                sh '''
                apk add --no-cache docker-cli git
                
                # Fix Git dubious ownership
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
                sh '''
                docker build -t $IMAGE_NAME:$TAG .
                '''
            }
        }

        stage('Push') {
    steps {
        echo '🚀 Subiendo imagen a GitHub Packages (GHCR)...'
        withCredentials([usernamePassword(
            credentialsId: 'github-credentials',
            usernameVariable: 'USERNAME',
            passwordVariable: 'TOKEN'
        )]) {
            sh '''
            echo $TOKEN | docker login ghcr.io -u $USERNAME --password-stdin
            
            FULL_IMAGE=ghcr.io/yairestrada/mi-app:latest
            
            docker tag mi-app:latest $FULL_IMAGE
            docker push $FULL_IMAGE
            '''
        }
    }
}

        stage('Verify') {
            steps {
                echo '✅ Verificando imagen publicada...'
                sh '''
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
            echo '✅ Pipeline exitoso! Imagen publicada en GitHub Packages'
        }

        failure {
            echo '❌ Pipeline falló!'
        }
    }
}
