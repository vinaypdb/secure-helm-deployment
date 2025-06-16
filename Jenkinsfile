pipeline {
    agent any

    environment {
        IMAGE_NAME = "sevenajay/mario"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📦 Cloning Repository..."
                git branch: 'main', url: 'https://github.com/vinaypdb/secure-helm-deployment.git'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "🔍 Scanning for vulnerabilities..."
                sh 'trivy fs . || true'
                sh 'trivy config . || true'
            }
        }

        stage('Docker Build & Push') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'

                echo "🚀 Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Helm Lint & Template') {
            steps {
                echo "🔍 Helm lint and template check..."
                sh 'helm lint charts/mario'
                sh 'helm template charts/mario'
            }
        }

        stage('Helm Deploy') {
            steps {
                echo "🚢 Deploying with Helm..."
                sh 'helm upgrade --install mario charts/mario'
            }
        }
    }
}



