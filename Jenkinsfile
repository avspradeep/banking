pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Add in Jenkins credentials
        DOCKERHUB_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        DOCKERHUB_PASS = "${DOCKERHUB_CREDENTIALS_PSW}"
        IMAGE_NAME = "fundme-app"
        IMAGE_TAG = "latest"
        REGISTRY = "docker.io/${DOCKERHUB_USER}"
        KUBECONFIG_CREDENTIALS = credentials('kubeconfig-creds') // Jenkins secret for kubeconfig file
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/avspradeep/banking.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                    sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Smoke Test') {
            steps {
                sh 'kubectl get pods -o wide'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Build/Deployment Failed!'
        }
    }
}
