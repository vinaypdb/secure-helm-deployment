pipeline {
    agent any

    environment {
        IMAGE_NAME = "vinaypdb/mario"
        IMAGE_TAG = "latest"
        CHART_DIR = "charts/mario"
        RELEASE_NAME = "mario-release"
        NAMESPACE = "mario-ns"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "🔧 Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo "🔍 Running Trivy scan using Docker container..."
                sh '''
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      aquasec/trivy:latest image $IMAGE_NAME:$IMAGE_TAG
                '''
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
                echo "📦 Generating Helm templates..."
                sh 'helm template $RELEASE_NAME $CHART_DIR'
            }
        }

        stage('Helm Deploy') {
            steps {
                echo "🚀 Deploying to Kubernetes with Helm..."
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

