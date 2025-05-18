pipeline {
    agent any

    environment {
        DOCKER_USER = 'fatou0409'
        BACKEND_IMAGE = "${DOCKER_USER}/profilapp-backend"
        FRONTEND_IMAGE = "${DOCKER_USER}/profilapp-frontend"
        MIGRATE_IMAGE = "${DOCKER_USER}/profilapp-migrate"
        SONARQUBE_URL = "http://localhost:9000"
        SONARQUBE_TOKEN = credentials('fafa')
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main', url: 'https://github.com/fatou0409/projet_fil_rouge.git'
            }
        }

        stage('Build des images') {
            steps {
                bat 'docker build -t %BACKEND_IMAGE%:latest ./Backend-main/odc'
                bat 'docker build -t %FRONTEND_IMAGE%:latest ./Frontend-main'
                bat 'docker build -t %MIGRATE_IMAGE%:latest ./Backend-main/odc'
            }
        }

        stage('Analyse SonarQube - Backend') {
            steps {
                dir('Backend-main/odc') {
                    withEnv(["SONAR_TOKEN=${SONARQUBE_TOKEN}"]) {
                        bat '''
                            "C:\\Users\\hp\\Desktop\\SonarScanner\\sonar-scanner\\bin\\sonar-scanner.bat" ^
                            -Dsonar.projectKey=backend ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONARQUBE_URL% ^
                            -Dsonar.login=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        stage('Analyse SonarQube - Frontend') {
            steps {
                dir('Frontend-main') {
                    withEnv(["SONAR_TOKEN=${SONARQUBE_TOKEN}"]) {
                        bat '''
                            "C:\\Users\\hp\\Desktop\\SonarScanner\\sonar-scanner\\bin\\sonar-scanner.bat" ^
                            -Dsonar.projectKey=frontend ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONARQUBE_URL% ^
                            -Dsonar.login=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        stage('Push des images sur Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'jenk', url: '']) {
                    retry(3) {
                        bat 'docker push %BACKEND_IMAGE%:latest'
                        bat 'docker push %FRONTEND_IMAGE%:latest'
                        bat 'docker push %MIGRATE_IMAGE%:latest'
                    }
                }
            }
        }

        stage('Terraform - Déploiement sur Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    withEnv(["KUBECONFIG=%KUBECONFIG_FILE%"]) {
                        dir('terraform') {
                            bat 'dir *.tf'
                            bat '"C:\\Users\\hp\\Desktop\\terraform_1.11.4_windows_amd64\\terraform.exe" init'
                            bat '"C:\\Users\\hp\\Desktop\\terraform_1.11.4_windows_amd64\\terraform.exe" plan -out=tfplan'
                            bat '"C:\\Users\\hp\\Desktop\\terraform_1.11.4_windows_amd64\\terraform.exe" apply -auto-approve tfplan'
                        }
                    }
                }
            }
        }
    }
}
