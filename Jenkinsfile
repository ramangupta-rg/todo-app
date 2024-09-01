pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'todo-app'   // Docker image name
        DOCKER_REGISTRY = 'ramangupta21/todo-app' // Replace with your Docker Hub username and image name
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Convert Windows path to Unix-style path if on Windows
                    def convertPathToUnix = { path ->
                        if (!isUnix()) {
                            def driveLetter = path.substring(0, 1).toLowerCase()
                            def withoutDrive = path.substring(2)
                            def unixPath = withoutDrive.replace('\\', '/')
                            return "/${driveLetter}${unixPath}"
                        }
                        return path
                    }

                    def workspace = env.WORKSPACE
                    def unixWorkspace = convertPathToUnix(workspace)

                    echo "Converted Unix Workspace Path: ${unixWorkspace}"

                    // Run tests using Docker directly
                    if (isUnix()) {
                        sh """
                        docker run --rm -v ${unixWorkspace}:/app -w /app ${DOCKER_IMAGE} python -m unittest discover
                        """
                    } else {
                        bat """
                        docker run --rm -v ${unixWorkspace}:/app -w /app ${DOCKER_IMAGE} python -m unittest discover
                        """
                    }
                }
            }
        }

        stage('Deploy') {
    steps {
        script {
            // Push the Docker image to Docker Hub
            docker.withRegistry('', '13bee838-94fb-4248-a117-3e3f07f246cc') {
                dockerImage.push("latest")
            }

            // Deploy the Docker container
            if (isUnix()) {
                sh """
                docker run -d -p 5000:5000 --name todo-app-deployment ${DOCKER_REGISTRY}:latest
                """
            } else {
                bat """
                docker run -d -p 5000:5000 --name todo-app-deployment ${DOCKER_REGISTRY}:latest
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
                if (isUnix()) {
                    // Unix/Linux clean up
                    sh "docker rmi ${DOCKER_IMAGE} || true"
                    sh "docker rmi ${DOCKER_REGISTRY}:latest || true"
                } else {
                    // Windows clean up
                    bat """
                    docker rmi ${DOCKER_IMAGE} 2>nul || echo Image not found, skipping cleanup.
                    docker rmi ${DOCKER_REGISTRY}:latest 2>nul || echo Image not found, skipping cleanup.
                    """
                }
            }
        }
    }
}
