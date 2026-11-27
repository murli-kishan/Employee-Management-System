pipeline {
	agent any

        environment {
            IMAGE_TAG = "${BUILD_NUMBER}"
            REPO_URL = "https://github.com/murli-kishan/Employee-Management-System.git"
        }

	
        stages {
	    stage('checkout') {
                steps {
                    git branch: 'main', url: "${REPO_URL}"
               	    
                }
            }
       
            stage('build Images') {
                steps {
                    sh 'docker compose build'
                }
            }
            
            stage('Docker Hub Login') {
                steps {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }

               }
            } 
            
            stage('push images') {
                steps {
                    sh 'docker push harryp0tter/employee-management-backend:${IMAGE_TAG}'
                    sh 'docker push harryp0tter/employee-management-frontend:${IMAGE_TAG}'
                }
            }
            
            stage('Deploy') {
                steps {
                    dir('ansible') {
                        sh 'ansible-playbook -i /etc/ansible/hosts deploy.yml --extra-vars "image_tag=${IMAGE_TAG}"'
                    }
                }
            }
    
        }
}
           
