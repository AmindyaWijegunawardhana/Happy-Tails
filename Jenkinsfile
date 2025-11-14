pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/AmindyaWijegunawardhana/Happy-Tails',
                    credentialsId: '2698a822-32d0-445d-b52f-e34b87034630'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '2698a822-32d0-445d-b52f-e34b87034630', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Build Containers') {
            steps {
                sh '''
                docker build -t amindya/backend-api:latest ./backend
                docker build -f ./frontend/Dockerfile -t amindya/react-app:latest ./frontend
                '''
            }
        }

        stage('Push Containers') {
            steps {
                sh '''
                docker push amindya/backend-api:latest
                docker push amindya/react-app:latest
                '''
            }
        }

        stage('Run Containers') {
            steps {
                sh '''
                docker stop backend || true
                docker rm backend || true
                docker stop frontend || true
                docker rm frontend || true

                docker run -d --name backend amindya/backend-api:latest
                docker run -d --name frontend -p 3000:3000 amindya/react-app:latest
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'docker ps -a'
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build failed!'
        }
    }
}
