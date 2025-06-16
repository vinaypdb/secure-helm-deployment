pipeline {
    agent any

    environment {
        IMAGE_NAME = "vinaypdb/mario"
        IMAGE_TAG = "latest"
        CHART_DIR = "charts/mario"
        RELEASE_NAME = "mario-release"
        NAMESPACE = "mario-ns"
        DOCKER_USER = credentials('docker-username')   // Jenkins credential ID
        DOCKER_PASS = credentials('docker-password')   // Jenkins credential ID
    }

    stages {
        stage('Docker Login') {
            steps {
                echo "🔐 Logging in to Docker Hub..."
                sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🔧 Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo "🔍 Running Trivy vulnerability scan..."
                // You can use --exit-code 1 to fail the build on HIGH/CRITICAL vulnerabilities
                sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Helm Lint') {
            steps {
                echo "🔍 Linting Helm chart..."
                sh 'helm lint $CHART_DIR'
            }
        }

        stage('Helm Template') {
            steps {
                echo "📦 Rendering Helm templates..."
                sh 'helm template $RELEASE_NAME $CHART_DIR'
            }
        }

        stage('Helm Deploy') {
            steps {
                echo "🚀 Deploying application with Helm..."
                sh 'helm upgrade --install $RELEASE_NAME $CHART_DIR --namespace $NAMESPACE --create-namespace'
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully!'
        }
        failure {
            echo '❌ CI/CD pipeline failed. Please check the logs.'
        }
    }
}

