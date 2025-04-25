pipeline {
    agent any

    environment {
        DOCKER_USER = 'fatou0409'
        BACKEND_IMAGE = "${DOCKER_USER}/profilapp-backend"
        FRONTEND_IMAGE = "${DOCKER_USER}/profilapp-frontend"
        MIGRATE_IMAGE = "${DOCKER_USER}/profilapp-migrate"
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/fatou0409/projet_fil_rouge.git'
            }
        }
        stage('Build des images') {
            steps {
                bat 'docker build -t $BACKEND_IMAGE:latest ./Backend-main/odc'
                bat 'docker build -t $FRONTEND_IMAGE:latest ./Frontend-main'
                bat 'docker build -t $MIGRATE_IMAGE:latest ./Backend-main/odc'
            }
        }

        stage('Push des images sur Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'jenk', url: '']) {
                    bat 'docker push $BACKEND_IMAGE:latest'
                    bat 'docker push $FRONTEND_IMAGE:latest'
                    bat 'docker push $MIGRATE_IMAGE:latest'
                }
            }
        }

        stage('Déploiement local avec Docker Compose') {
            steps {
                bat '''
                    docker-compose down || true
                    docker-compose pull
                    docker-compose up -d --build
                '''
            }
        }
    }

        }
}
