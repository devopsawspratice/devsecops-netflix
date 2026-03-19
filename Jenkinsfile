pipeline {
    agent any

    environment {
        DOCKER_HUB = 'veeradockerhub'
        IMAGE_NAME = 'netflix-clone'
        IMAGE_TAG  = 'latest'
        KUBE_DEPLOY_PATH = 'k8s/deployment.yaml'
        KUBE_SERVICE_PATH = 'k8s/service.yaml'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/veeraprasadkoduri-cmd/devsecops-netflix.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                script {
                    if (credentials('docker-hub-pass')) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-pass', 
                                                          usernameVariable: 'DOCKER_USER', 
                                                          passwordVariable: 'DOCKER_PASS')]) {
                            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            sh "docker push ${DOCKER_HUB}/${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    } else {
                        echo "⚠️ Docker Hub credentials not found. Skipping push."
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${KUBE_DEPLOY_PATH}"
                sh "kubectl apply -f ${KUBE_SERVICE_PATH}"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods -o wide"
                sh "kubectl get svc -o wide"
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
