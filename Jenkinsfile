pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ramangupta21/todo-app"
        DOCKER_TAG = "V1"
        KUBECONFIG_CREDENTIALS_ID = 'your-kubeconfig-credentials-id'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ramangupta-rg/todo-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f kubernetes/'
                }
            }
        }
    }
}
