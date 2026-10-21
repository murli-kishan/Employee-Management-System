pipeline {
    agent any

    environment {
        CRED_ID = 'dockerhub-credentials'
        K8S_CRED_ID = 'kind-kubeconfig'
        PATH = "C:\\Users\\dmurl\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;C:\\Users\\dmurl\\AppData\\Local\\Microsoft\\WinGet\\Links;${env.PATH}"
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

        stage('Deploy with Helm') {
            steps {
                dir('ems-k8s') {
                    deleteDir()

                    git branch: 'main', 
                        url: 'https://github.com/murlikishan8-create/ems-k8s.git'
                
                    withCredentials([
                        file(
                            credentialsId: env.K8S_CRED_ID,
                            variable: 'KUBECONFIG'
                       )
                    ]) {
                        bat 'helm version --short'
                        bat 'kubectl get nodes'

                        bat 'helm upgrade --install ems .\\ems --wait --timeout 5m'
                        
                        bat 'kubectl rollout restart deployment/backend'
                        bat 'kubectl rollout restart deployment/frontend'

                        bat 'kubectl rollout status deployment/backend --timeout=5m'
                        bat 'kubectl rollout status deployment/frontend --timeout=5m'

                        bat 'kubectl get pods,pvc'
                    }
                }
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
    }
}