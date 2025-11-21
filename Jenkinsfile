pipeline {
    agent any

    environment {
        IMAGE = "docker.io/greeshma258/jenkins:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image using Podman') {
            steps {
                sh """
                    podman build -t ${IMAGE} .
                """
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh """
                        podman login docker.io -u ${USER} -p ${PASS}
                        podman push ${IMAGE}
                    """
                }
            }
        }

        stage('Deploy to RKE2 Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/welcome-app-deployment -n demo
                """
            }
        }

        stage('Verify Pods') {
            steps {
                sh "kubectl get pods -n demo"
            }
        }

    }
}
