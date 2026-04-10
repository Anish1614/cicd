pipeline {
    agent any

    environment {
        // Define Docker registry repository
        DOCKER_IMAGE = 'devopsjenkins1614/cicd'
        // Hardcoded credentials for testing purposes
        DOCKER_USER = 'devopsjenkins1614'
        DOCKER_PASS = 'Anish@399'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from public github repo
                git branch: 'main', url: 'https://github.com/Anish1614/cicd'
            }
        }

        stage('Test') {
            steps {
                // Jenkins host (Ubuntu 24.04) has python3 installed natively
                // Use a virtual environment to avoid PEP 668 restrictions on global pip
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pytest
                '''
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Generate a timestamp for the image tag
                    env.IMAGE_TAG = sh(script: "date +'%Y%m%d%H%M%S'", returnStdout: true).trim()
                    echo "Building Docker image: ${env.DOCKER_IMAGE}:${env.IMAGE_TAG}"
                    
                    // Build the docker image using standard docker shell commands
                    sh 'docker build -t "${DOCKER_IMAGE}:${IMAGE_TAG}" .'
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub using standard shell commands"
                    
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        # Push the timestamped tag
                        docker push "${DOCKER_IMAGE}:${IMAGE_TAG}"
                        
                        # Tag as latest and push
                        docker tag "${DOCKER_IMAGE}:${IMAGE_TAG}" "${DOCKER_IMAGE}:latest"
                        docker push "${DOCKER_IMAGE}:latest"
                    '''
                }
            }
        }
    }

    // post {
    //     always {
    //         // Clean workspace after pipeline completion
    //         cleanWs()
    //         // Clean up left over images locally
    //         script {
    //             try {
    //                 sh 'docker rmi "${DOCKER_IMAGE}:${IMAGE_TAG}" || true'
    //                 sh 'docker rmi "${DOCKER_IMAGE}:latest" || true'
    //             } catch (Exception e) {
    //                 echo "Skipping docker rmi cleanup"
    //             }
    //         }
    //     }
    // }
}
