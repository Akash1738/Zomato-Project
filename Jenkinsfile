pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar'
        IMAGE_NAME   = "Akash/zomato-app:latest"
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
                    url: 'https://https://github.com/Akash1738/Zomato-Project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Zomato-App \
                    -Dsonar.projectKey=zomato-app
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false,
                                  credentialsId: 'sonar-token'
            }
        }

//       stage('OWASP Dependency Check') {
//     steps {
//         sh '''
//         /opt/dependency-check/bin/dependency-check.sh \
//         --scan . \
//         --format XML \
//         --out dependency-check-report
//         '''
//     }
// }



        stage('Trivy FileSystem Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
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
                        docker tag zomato-app sagarbarve/zomato-app:latest
                        docker push sagarbarve/zomato-app:latest
                        '''
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image sagarbarve/zomato-app:latest'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f zomato-container || true
                docker run -d \
                --name zomato-container \
                -p 3000:3000 \
                sagarbarve/zomato-app:latest
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
