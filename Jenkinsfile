pipeline {
    agent any

    environment {
        DOCKER_HUB = 'awsdevopspratice'
        IMAGE_NAME = 'netflix-clone'
        IMAGE_TAG  = 'latest'
        KUBE_DEPLOY_PATH = 'Kubernetes/deployment.yml'
        KUBE_SERVICE_PATH = 'Kubernetes/service.yml'
        KUBECONFIG_PATH = '/var/lib/jenkins/.kube/config'  // Explicit kubeconfig
        RECIPIENTS = 'sharan0434@gmail.com'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/devopsawspratice/devsecops-netflix.git', branch: 'main'
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
                    credentialsId: 'docker-creds',
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
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    kubectl apply -f ${KUBE_DEPLOY_PATH}
                    kubectl apply -f ${KUBE_SERVICE_PATH}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    echo 'Waiting for pods to be ready...'
                    kubectl rollout status deployment/netflix-app --timeout=120s
                    kubectl get pods -o wide
                    kubectl get svc -o wide
                """
            }
        }
    }

    post {
       success {
        echo "✅ Pipeline completed successfully!"
        emailext (
            to: "${RECIPIENTS}",
            subject: "Jenkins Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """The pipeline completed successfully!
            Job: ${env.JOB_NAME}
            Build: ${env.BUILD_NUMBER}
            Check console output at: ${env.BUILD_URL}"""
        )
    }
       
    }
}
