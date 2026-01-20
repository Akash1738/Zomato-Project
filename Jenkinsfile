pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        IMAGE_NAME = "akash1738/zomato-app:latest"
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
                    url: 'https://github.com/Akash1738/Zomato-Project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'   // reproducible installs using package-lock.json
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t zomato-app .'
            }
        }

        stage('Tag & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-password') {
                        sh '''
                        docker tag zomato-app ${IMAGE_NAME}
                        docker push ${IMAGE_NAME}
                        '''
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f zomato-container || true
                docker run -d \
                  --name zomato-container \
                  -p 3000:3000 \
                  ${IMAGE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Zomato CI/CD Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Zomato CI/CD Pipeline Failed"
        }
    }
}
