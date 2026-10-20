pipeline {
    agent any

     
    environment {
        CRED_ID = 'dockerhub-credentials'

        PATH = "C:\\Users\\dmurl\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                bat 'docker --version'
                bat 'docker build -t ems-backend ./server'
                bat 'docker build -t ems-frontend ./client'
            }
        }

        // stage('Security Scan') {
        //     steps {
        //         // Trivy container scans the images for critical CVEs
        //         // Prepend 'bash' so Windows cmd can execute the .sh script
        //         bat 'bash ./scan.sh ems-backend'
        //         bat 'bash ./scan.sh ems-frontend'
        //     }
        // }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.CRED_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'powershell -NoProfile -Command "$env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin"'

                    bat 'docker tag ems-backend %DOCKER_USER%/ems-backend:latest'
                    bat 'docker tag ems-frontend %DOCKER_USER%/ems-frontend:latest'

                    bat 'docker push %DOCKER_USER%/ems-backend:latest'
                    bat 'docker push %DOCKER_USER%/ems-frontend:latest'
                }
            }
        }
<<<<<<< HEAD

        stage('Deploy to Kubernetes') {
            steps {
                dir('ems-k8s') {
                    withCredentials([
                        file(
                            credentialsID: env.k8s_CRED_ID,
                            variable: 'KUBECONFIG'
                        )
                    ]) {
                        bat 'helm upgrade --install ems . \\ems --set-string backend.image.tag=%IMAGE_TAG% --set-string frontend.image.tag=%IMAGE_TAG% --wait --timeout 5m'

                        bat 'kubectl rollout status deployment/backend --timeout=5m'
                        bat 'kubectl rollout status deployment/frontend --timeout=5m'

                        bat 'kubectl get pods'
                    }
                }
            }
        }
=======
>>>>>>> 6408ffc29cfa75c27b47985ccab7d461c29ade03
    }
    
    post {
        always {
            bat 'docker logout'
        }
    }
}


