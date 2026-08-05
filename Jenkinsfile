pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/starbucks:latest"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Anshika7820/starbucks.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh 'node -v'
                sh 'npm -v'
                sh 'git --version'
                sh 'docker --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                sh '''
                export CI=false
                export NODE_OPTIONS="--max-old-space-size=2048"
                npm run build
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image $IMAGE_NAME > trivy-report.txt || true
                '''
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh '''
                docker stop starbucks || true
                docker rm starbucks || true

                docker run -d \
                  --name starbucks \
                  -p 3000:3000 \
                  $IMAGE_NAME
                '''
            }
        }

    }

    post {

        success {
            echo 'Build completed successfully!'
        }

        failure {
            echo 'Build failed!'
        }

        always {
            archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
            cleanWs()
        }
    }
}
