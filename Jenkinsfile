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
                sh 'sh ./mvnw clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            environment {
                DOCKER_USER = credentials('dockerhub-creds').username
                DOCKER_PASS = credentials('dockerhub-creds').password
            }
            steps {
                sh '''
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker push ${DOCKER_IMAGE}
                    docker logout
                '''
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
