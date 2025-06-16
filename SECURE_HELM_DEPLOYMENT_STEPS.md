# Secure Super Mario Game Deployment with Helm-based CI/CD on Minikube

## 📌 Project Objective

Deploy the Super Mario game Docker image (`sevenajay/mario`) using a CI/CD pipeline powered by Jenkins, Docker, Trivy, Helm, and Kubernetes (Minikube). This project ensures secure, automated deployment using best practices.

---

## 🧱 Prerequisites

* OS: Ubuntu (local machine)
* Tools installed:

  * Docker
  * Minikube
  * kubectl
  * Helm
  * Trivy
  * Jenkins (run using `.war` file)
* GitHub repository: [https://github.com/vinaypdb/secure-helm-deployment](https://github.com/vinaypdb/secure-helm-deployment)

---

## 🪜 Step-by-Step Project Execution

### Step 1: Create GitHub Repository

* Created a repo `secure-helm-deployment`.
* Added the following structure:

  ```bash
  ├── charts
  │   └── mario
  │       ├── Chart.yaml
  │       ├── values.yaml
  │       └── templates/
  └── Jenkinsfile
  ```
* Base Dockerfile from `sevenajay/mario`.

---

### Step 2: Create Jenkinsfile

Defined the CI/CD pipeline stages:

```groovy
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
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }
    stage('Push Docker Image') {
      steps {
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
        sh 'trivy image $IMAGE_NAME:$IMAGE_TAG'
      }
    }
    stage('Helm Lint') {
      steps {
        sh 'helm lint $CHART_DIR'
      }
    }
    stage('Helm Template') {
      steps {
        sh 'helm template $RELEASE_NAME $CHART_DIR'
      }
    }
    stage('Helm Deploy') {
      steps {
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
```

---

### Step 3: Configure Jenkins

* Downloaded `jenkins.war` and launched with:

  ```bash
  export JENKINS_HOME=$HOME/jenkins-home
  java -jar $HOME/jenkins.war
  ```
* Accessed Jenkins at `http://localhost:8080`
* Set up new admin user and unlocked Jenkins
* Installed suggested plugins
* Created new pipeline project and configured GitHub repo URL

---

### Step 4: Setup DockerHub Credentials in Jenkins

* Go to: `Manage Jenkins` > `Credentials` > `(global)` > `Add Credentials`
* Type: `Username with password`
* ID: `dockerhub-creds`
* Username: `vinaypdb`
* Password: DockerHub PAT or password

---

### Step 5: Setup Minikube

* Start Minikube:

  ```bash
  minikube start --driver=docker
  ```
* Enable ingress and dashboard:

  ```bash
  minikube addons enable ingress
  minikube addons enable dashboard
  ```

---

### Step 6: Install Trivy

* Installed Trivy via script or package manager
* Tested scan:

  ```bash
  trivy image vinaypdb/mario:latest
  ```

---

### Step 7: Run Jenkins Job

* Triggered build manually from Jenkins UI
* Verified each stage:

  * Docker build & push
  * Trivy scan
  * Helm lint & template
  * Helm deployment

---

### Step 8: Verify Deployment

```bash
kubectl get all -n mario-ns
minikube service mario-release -n mario-ns
```

Or:

```bash
minikube ip  # e.g., 192.168.49.2
Access at: http://192.168.49.2:<NodePort>
```

---

## ✅ Final Output

Super Mario game successfully deployed on Minikube via secure Jenkins CI/CD pipeline with Docker, Helm, and Trivy.

---

## 📘 Repository

[GitHub - vinaypdb/secure-helm-deployment](https://github.com/vinaypdb/secure-helm-deployment)

