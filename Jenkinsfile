pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'todo-app'   // Define your Docker image name
        DOCKER_REGISTRY = 'ramangupta21/todo-app' // Replace with your Docker Hub username and image name
    }

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Convert the Windows path to Unix-style path for Docker
                    def unixWorkspace = "${env.WORKSPACE}".replaceAll('C:/', '/c/').replace('\\', '/')

                    // Run tests inside the Docker container
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
                        // Push the Docker image to the registry
                        dockerImage.push("latest")

                        // Run the Docker container in detached mode
                        sh """
                        docker run -d -p 5000:5000 ${DOCKER_REGISTRY}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up Docker images to save space
                sh "docker rmi ${DOCKER_IMAGE} || true"
                sh "docker rmi ${DOCKER_REGISTRY}:latest || true"
            }
        }
    }
}
