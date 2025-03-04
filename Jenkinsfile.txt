pipeline {
    agent any
    
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('myrepo/myapp:latest')
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: DOCKER_CREDENTIALS_ID, url: 'https://index.docker.io/v1/']) {
                    sh 'docker push myrepo/myapp:latest'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: KUBE_CONFIG_CREDENTIALS_ID]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}

