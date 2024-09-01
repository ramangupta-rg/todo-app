pipeline {
    agent any

    environment {
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("todo-app")
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    dockerImage.inside("-v ${WORKSPACE}:/app -w /app") {
                        sh 'python -m unittest discover'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        dockerImage.push("ramanguptarg21/todo-app:latest")
                        sh "docker run -d -p 5000:5000 ramanguptarg21/todo-app:latest"
                    }
                }
            }
        }
    }
}
