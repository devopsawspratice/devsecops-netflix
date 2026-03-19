pipeline {
    agent any

    environment {
        IMAGE_NAME = "veeradockerhub/netflix-clone"
        IMAGE_TAG  = "latest"
        KUBE_CONFIG = "/root/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/veeraprasadkoduri-cmd/devsecops-netflix.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                script {
                    if (credentialsExists('docker-hub-pass')) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-pass', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                            sh 'echo $PASS | docker login -u $USER --password-stdin'
                            sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    } else {
                        echo "⚠️ Docker Hub credentials not found. Skipping push."
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods -A'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Succeeded!"
        }
        failure {
            echo "❌ Pipeline Failed!"
        }
    }
}

// Helper function to check if credentials exist
def credentialsExists(id) {
    try {
        def c = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
            com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class,
            Jenkins.instance,
            null,
            null
        ).find { it.id == id }
        return c != null
    } catch (e) {
        return false
    }
}
