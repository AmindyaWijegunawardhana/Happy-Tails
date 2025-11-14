def DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials' // ID you set in Step 3
def DOCKER_USER = 'YOUR_USERNAME' // Your Docker Hub username
def BACKEND_IMAGE = "${DOCKER_USER}/backend-api"
def FRONTEND_IMAGE = "${DOCKER_USER}/react-app"
def BUILD_TAG = sh(returnStdout: true, script: 'echo ${BUILD_NUMBER}').trim() // Uses Jenkins build number as tag

pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                // Replace with your Git repository URL
                git url: 'YOUR_GIT_REPOSITORY_URL'
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                // Use the credentials binding plugin to inject credentials
                withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}"
                }
            }
        }
        
        stage('Build & Push Backend') {
            steps {
                echo 'Building backend image...'
                // Build the backend image, tagging it with the build number
                sh "docker build -t ${BACKEND_IMAGE}:${BUILD_TAG} -t ${BACKEND_IMAGE}:latest ./backend"
                
                echo 'Pushing backend image...'
                // Push both the numbered tag and the 'latest' tag
                sh "docker push ${BACKEND_IMAGE}:${BUILD_TAG}"
                sh "docker push ${BACKEND_IMAGE}:latest"
            }
        }
        
        stage('Build & Push Frontend') {
            steps {
                echo 'Building frontend image...'
                // Build the frontend image from the root, passing context for multi-stage build
                sh "docker build -f ./frontend/Dockerfile -t ${FRONTEND_IMAGE}:${BUILD_TAG} -t ${FRONTEND_IMAGE}:latest ."
                
                echo 'Pushing frontend image...'
                sh "docker push ${FRONTEND_IMAGE}:${BUILD_TAG}"
                sh "docker push ${FRONTEND_IMAGE}:latest"
            }
        }
        
        stage('Cleanup') {
            steps {
                // Optional: remove local images after successful push to save disk space
                sh "docker rmi -f ${BACKEND_IMAGE}:latest"
                sh "docker rmi -f ${FRONTEND_IMAGE}:latest"
            }
        }
    }
}