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

        stage('Test Kubeconfig') {
            steps {
                withKubeConfig(credentialsId: 'rke2-kubeconfig') {
                    sh "kubectl get nodes"
                }
            }
        }

        stage('Deploy to RKE2 Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'rke2-kubeconfig') {
                    sh """
                        kubectl apply -f k8s/deployment.yaml -n demo
                        kubectl apply -f k8s/service.yaml -n demo
                        kubectl rollout status deployment/welcome-app-deployment -n demo
                    """
                }
            }
        }

        stage('Verify Pods') {
            steps {
                withKubeConfig(credentialsId: 'rke2-kubeconfig') {
                    sh "kubectl get pods -n demo"
                }
            }
        }

    }
}
