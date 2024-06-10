pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aycaakyol3/flask-app-web:latest'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // Update with your Docker credentials ID
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig' // Update with your kubeconfig credentials ID
        SONARQUBE_SCANNER = 'SonarQube'
        SONARQUBE_TOKEN = 'sonarqube-token' // Update with your SonarQube token ID
        // DOCKER_HOME = tool name: 'myDocker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        PATH = "/opt/homebrew/bin:${env.PATH}"
        SONAR_HOST_URL = 'http://sonarqube:9000'
        KUBECONFIG = '/root/.kube/config'
    }

    stages {
        // stage('Initialize') {
        //     steps {
        //         script {
        //             env.PATH = "${DOCKER_HOME}/bin:${env.PATH}"
        //         }
        //     }
        // }

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
                        sh 'sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONARQUBE_TOKEN'
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
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f kubernetes/deployment.yaml'
                        sh 'kubectl apply -f kubernetes/service.yaml'
                    }
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
