pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pradeeproy66/banking:latest"
        DOCKER_USER  = "pradeeproy66"
        DOCKER_PASS  = "dckr_pat_Wvga4VUsa6SAEatxyphfK4cvk_g"
        KUBE_CONFIG  = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/avspradeep/banking.git', branch: 'master'
            }
        }

        stage('Build & Test') {
            steps {
                sh './mvnw clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${DOCKER_IMAGE}
                    docker logout
                '''
            }
        }

        stage('Test kubectl') {
            steps {
                sh 'which kubectl'
                sh 'kubectl version --client'
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
