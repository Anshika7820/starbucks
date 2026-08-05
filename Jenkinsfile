pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
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
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                export NODE_OPTIONS="--max-old-space-size=2048"
                export CI=false
                npm run build
                '''
            }
        }

        stage('Test Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo 'Build completed successfully!'
        }

        failure {
            echo 'Build failed!'
        }
    }
}
