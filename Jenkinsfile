pipeline {
    agent any

    environment {
        IMAGE_NAME = 'veeradockerhub/netflix-clone:latest'
        DOCKER_CRED = credentials('docker-hub-pass') // Docker Hub credentials
        KUBE_CONFIG = '/root/.kube/config'          // Path to kubeconfig on Jenkins agent
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
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

        stage('Build Project') {
            steps {
                sh 'yarn build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-pass', 
                                                 passwordVariable: 'PASS', 
                                                 usernameVariable: 'USER')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Set kubeconfig for kubectl
                    env.KUBECONFIG = "${KUBE_CONFIG}"

                    // Deploy manifests
                    sh '''
                        kubectl apply -f Kubernetes/ --validate=false
                        
                        # Wait for deployment rollout
                        kubectl rollout status deployment/netflix-app --timeout=120s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get svc
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
