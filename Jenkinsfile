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
                    // Convert the Windows path to a Unix-style path
                    def unixWorkspace = "${env.WORKSPACE}".replaceAll('C:/', '/c/').replace('\\', '/')

                    // Mount the workspace as a volume and set the working directory
                    dockerImage.inside("-v ${unixWorkspace}:/app -w /app") {
                        sh 'python -m unittest discover'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        dockerImage.push("ramangupta21/todo-app:latest")
                        sh "docker run -d -p 5000:5000 ramangupta21/todo-app:latest"
                    }
                }
            }
        }
    }
}
