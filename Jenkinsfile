pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'asihf'
        IMAGE_NAME      = 'healthcare-portal'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% ."
                    bat "docker build -t %DOCKER_HUB_USER%/%IMAGE_NAME%:latest ."
                }
            }
        }
        
        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    bat "echo %DOCKER_PASSWORD%| docker login -u %DOCKER_USERNAME% --password-stdin"
                    bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%"
                    bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:latest"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    powershell """
                        (Get-Content deployment.yaml) -replace 'image: .*', "image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}" | Set-Content deployment.yaml
                    """
                    powershell """
                        kubectl apply -f deployment.yaml --kubeconfig="C:\\Users\\batch1.VITUNIVERSITY\\.kube\\config"
                        kubectl rollout status deployment/healthcare-portal-deployment --kubeconfig="C:\\Users\\batch1.VITUNIVERSITY\\.kube\\config"
                    """
                }
            }
        }
    }
    
    post {
        failure {
            script {
                echo "Deployment unhealthy! Reverting to the last working application tag..."
                powershell """
                    kubectl rollout undo deployment/healthcare-portal-deployment --kubeconfig="C:\\Users\\batch1.VITUNIVERSITY\\.kube\\config"
                """
            }
        }
    }
}
