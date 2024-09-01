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
                    def workspacePath = env.WORKSPACE.replace('\\', '/')
                    echo "Converted Unix Workspace Path: ${workspacePath}"

                    dockerImage.inside("-v ${workspacePath}:/app -w /app") {
                        sh 'python -m unittest discover -s tests'
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                echo 'Deploying the application...'
                // Add your deployment steps here
            }
        }
    }

    post {
        always {
            script {
                sh 'docker rmi todo-app || echo Image not found, skipping cleanup.'
            }
        }
    }
}
