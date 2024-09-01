pipeline {
    agent any

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
                    dockerImage.inside {
                        sh 'python -m unittest discover'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    dockerImage.push("ramangupta21/todo-app:latest")
                    sh "docker run -d -p 5000:5000 ramangupta21/todo-app:latest"
                }
            }
        }
    }
}
