pipeline {
    agent any

    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
        KUBECONFIG_PATH = "/var/jenkins_home/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/abhinav195/CodeRank.git'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build -t coderank-auth -f coderank-auth/Dockerfile .
                    docker build -t coderank-gateway -f coderank-gateway/Dockerfile .
                    docker build -t coderank-problem -f coderank-problem/Dockerfile .
                    docker build -t coderank-submission -f coderank-submission/Dockerfile .
                    docker build -t coderank-execution -f coderank-execution/Dockerfile .
                    docker build -t coderank-result-processor -f coderank-result-processor/Dockerfile .
                '''
            }
        }

        stage('Deploy Infra') {
            steps {
                sh '''
                    kubectl apply -f k8s/infra/postgres.yaml
                    kubectl apply -f k8s/infra/redis.yaml
                    kubectl apply -f k8s/infra/kafka.yaml
                    kubectl apply -f k8s/infra/observability.yaml
                '''
            }
        }

        stage('Deploy App') {
            steps {
                sh 'kubectl apply -f k8s/app/app-services.yaml'
            }
        }

        stage('Rollout Restart') {
            steps {
                sh '''
                    kubectl rollout restart deployment coderank-auth
                    kubectl rollout restart deployment coderank-gateway
                    kubectl rollout restart deployment coderank-problem
                    kubectl rollout restart deployment coderank-submission
                    kubectl rollout restart deployment coderank-execution
                    kubectl rollout restart deployment coderank-result-processor
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'kubectl get pods -A'
            }
        }
    }

    post {
        success {
            echo 'CodeRank deployment pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed — check stage logs above.'
        }
    }
}