pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        IMAGE_NAME = "anshi1008/starbucks"
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
                sh 'trivy --version'
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
                    export NODE_OPTIONS="--max-old-space-size=2048"
                    export CI=false
                    npm run build
                '''
            }
        }

        stage('SonarCloud Scan') {

            environment {
                SCANNER_HOME = tool 'sonar-scanner'
                SONAR_SCANNER_OPTS = "-Xmx1024m"
                NODE_OPTIONS = "--max-old-space-size=2048"
            }

            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                        export NODE_OPTIONS="--max-old-space-size=2048"
                        export SONAR_SCANNER_OPTS="-Xmx1024m"

                        $SCANNER_HOME/bin/sonar-scanner
                    '''
                }
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                    trivy fs \
                    --severity HIGH,CRITICAL \
                    --format table \
                    .
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --format table \
                    $IMAGE_NAME:latest
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop starbucks || true
                    docker rm starbucks || true

                    docker run -d \
                    --name starbucks \
                    -p 3000:3000 \
                    $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {

        success {
            echo "Pipeline executed successfully!"
        }

        failure {
            echo "Pipeline execution failed!"
        }

        always {
            cleanWs()
        }
    }
}
