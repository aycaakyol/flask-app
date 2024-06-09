pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aycaakyol3/flask-app-web:latest'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // Update with your Docker credentials ID
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig' // Update with your kubeconfig credentials ID
        SONARQUBE_SCANNER = 'SonarQube'
        SONARQUBE_TOKEN = 'sonarqube-token' // Update with your SonarQube token ID
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE_SCANNER) {
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    sh 'trivy image $DOCKER_IMAGE'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: KUBECONFIG_CREDENTIALS_ID]) {
                    sh 'kubectl apply -f kubernetes/deployment.yaml'
                    sh 'kubectl apply -f kubernetes/service.yaml'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
