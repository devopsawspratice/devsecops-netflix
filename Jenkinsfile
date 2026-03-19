pipeline {
    agent any

    environment {
        DOCKER_HUB = 'veeradockerhub'
        IMAGE_NAME = 'netflix-clone'
        IMAGE_TAG  = 'latest'
        KUBE_DEPLOY_PATH = 'Kubernetes/deployment.yml'
        KUBE_SERVICE_PATH = 'Kubernetes/service.yml'
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
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',  // <- Corrected ID
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS')]) {
                    
                    sh '''
                        echo "Logging into Docker Hub..."
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_HUB}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f ${KUBE_DEPLOY_PATH}
                    kubectl apply -f ${KUBE_SERVICE_PATH}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo 'Waiting for pods to be ready...'
                    kubectl rollout status deployment/netflix-app --timeout=120s
                    kubectl get pods -o wide
                    kubectl get svc -o wide
                '''
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
