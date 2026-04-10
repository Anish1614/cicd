pipeline {
    agent any

    environment {
        // Define Docker registry repository
        DOCKER_IMAGE = 'devopsjenkins1614/cicd'
        
        // Ensure you have configured Docker Hub credentials in Jenkins with this ID
        // Dashboard > Manage Jenkins > Credentials > System > Global credentials > Add Credentials
        // Kind: Username with password
        // ID: dockerhub-credentials
        DOCKER_REGISTRY_CREDENTIALS = 'dockerhub-credentials' 
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from source control
                checkout scm
            }
        }

        stage('Test') {
            // Run tests inside a Python Docker container so the Jenkins host doesn't need Python installed
            agent {
                docker { 
                    image 'python:3.11-slim' 
                    reuseNode true
                }
            }
            steps {
                // Determine if this is running on Windows (bat) or Linux (sh) for the host
                script {
                    isUnix() ? sh('pip install -r requirements.txt && pytest') : bat('pip install -r requirements.txt && pytest')
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${env.BUILD_ID}"
                    // Build the docker image
                    customImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub"
                    docker.withRegistry('', DOCKER_REGISTRY_CREDENTIALS) {
                        // Push the image with the build ID tag
                        customImage.push()
                        // Push the latest tag
                        customImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean workspace after pipeline completion
            cleanWs()
            // Clean up left over images locally
            script {
                try {
                    isUnix() ? sh("docker rmi ${DOCKER_IMAGE}:${env.BUILD_ID} || true") : bat("docker rmi ${DOCKER_IMAGE}:${env.BUILD_ID} || exit 0")
                } catch (Exception e) {
                    echo "Skipping docker rmi cleanup"
                }
            }
        }
    }
}
