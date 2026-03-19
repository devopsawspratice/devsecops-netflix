pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "veeradockerhub"
        DOCKER_HUB_PASS = credentials('docker-hub-pass') // Your Jenkins Docker Hub password stored as credential
        IMAGE_NAME = "netflix-clone"
        K8S_DIR = "Kubernetes"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/veeraprasadkoduri-cmd/devsecops-netflix.git', branch: 'main'
            }
        }

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Project & Docker Image') {
            steps {
                // Build the project
                sh 'yarn build'

                // Build Docker image
                sh "docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:latest ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-pass', variable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u veeradockerhub --password-stdin
                    docker push veeradockerhub/netflix-clone:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes YAMLs
                    if (fileExists(K8S_DIR)) {
                        sh """
                        kubectl apply -f $K8S_DIR/
                        kubectl rollout status deployment/netflix-app
                        """
                    } else {
                        error "Kubernetes folder not found!"
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                kubectl get pods
                kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Completed Successfully!"
        }
        failure {
            echo "❌ Pipeline Failed!"
        }
    }
}
