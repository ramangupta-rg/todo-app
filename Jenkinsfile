pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'todo-app'
        DOCKER_REGISTRY = 'ramangupta21/todo-app'
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
                    if (isUnix()) {
                        sh "docker stop todo-app-deployment || true"
                        sh "docker rm todo-app-deployment || true"
                    } else {
                        bat "docker stop todo-app-deployment 2>nul || echo No existing container to stop."
                        bat "docker rm todo-app-deployment 2>nul || echo No existing container to remove."
                    }

                    docker.withRegistry('', '13bee838-94fb-4248-a117-3e3f07f246cc') {
                        dockerImage.push("latest")
                    }

                    echo "Running container..."
                    if (isUnix()) {
                        sh """
                        docker run -d -p 5000:5000 --name todo-app-deployment ${DOCKER_REGISTRY}:latest
                        """
                    } else {
                        bat """
                        docker run -d -p 5000:5000 --name todo-app-deployment ${DOCKER_REGISTRY}:latest
                        """
                    }

                    echo "Checking if container is running..."
                    def isRunning
                    if (isUnix()) {
                        isRunning = sh(returnStatus: true, script: "docker ps | grep todo-app-deployment")
                    } else {
                        isRunning = bat(returnStatus: true, script: "docker ps | findstr todo-app-deployment")
                    }

                    if (isRunning != 0) {
                        error "Deployment failed: The Docker container did not start."
                    } else {
                        echo "Deployment successful: The Docker container is running."
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (isUnix()) {
                    sh "docker rmi ${DOCKER_IMAGE} || true"
                    sh "docker rmi ${DOCKER_REGISTRY}:latest || true"
                } else {
                    // Use a different approach to handle potential errors gracefully
                    bat """
                    docker rmi ${DOCKER_IMAGE} 2>nul || echo Image not found, skipping cleanup.
                    docker rmi ${DOCKER_REGISTRY}:latest 2>nul || echo Image not found, skipping cleanup.
                    """
                }
            }
        }
    }
}
