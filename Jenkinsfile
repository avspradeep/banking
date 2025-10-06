pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pradeeproy66/banking:latest"
        KUBE_CONFIG = credentials('kubeconfig-creds')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/avspradeep/banking.git', branch: 'master'
            }
        }

        stage('Build & Test') {
            steps {
                // Ensure mvnw runs with sh explicitly
                sh 'sh ./mvnw clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(["KUBECONFIG=${KUBE_CONFIG}"]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and Deployment Successful!"
        }
        failure {
            echo "❌ Build/Deployment Failed!"
        }
    }
}
