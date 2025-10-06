pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pradeeproy66/banking:latest" // your DockerHub username
        KUBE_CONFIG = credentials('kubeconfig-creds')  // upload your kubeconfig in Jenkins credentials
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/avspradeep/banking.git', branch: 'master'
            }
        }

        stage('Build & Test') {
            steps {
                // Use sh explicitly to avoid permission issues with mvnw
                sh 'sh ./mvnw clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(["KUBECONFIG=$KUBE_CONFIG"]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        success {
