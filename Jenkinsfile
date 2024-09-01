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
                    // Function to convert Windows path to Unix-style path
                    def convertPathToUnix = { path ->
                        // Extract drive letter and convert to lowercase
                        def driveLetter = path.substring(0,1).toLowerCase()
                        // Remove drive letter and colon
                        def withoutDrive = path.substring(2)
                        // Replace backslashes with forward slashes
                        def unixPath = withoutDrive.replace('\\', '/')
                        // Prepend with /c or appropriate drive letter
                        return "/${driveLetter}${unixPath}"
                    }

                    def workspace = env.WORKSPACE
                    def unixWorkspace = convertPathToUnix(workspace)

                    echo "Converted Unix Workspace Path: ${unixWorkspace}"

                    // Run tests using Docker directly
                    sh """
                    docker run --rm -v ${unixWorkspace}:/app -w /app ${DOCKER_IMAGE} python -m unittest discover
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    docker.withRegistry('', 'docker-hub-credentials') {
                        dockerImage.push("latest")
                    }

                    // Deploy the Docker container
                    sh """
                    docker run -d -p 5000:5000 ${DOCKER_REGISTRY}:latest
                    """
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
