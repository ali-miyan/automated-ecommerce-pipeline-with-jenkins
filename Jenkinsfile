pipeline {
    agent any

    environment {
        REGISTRY = 'hub.docker.com'
        REPO = 'alimiyn/ecommerce'
        IMAGE = 'alimiyn/ecommerce:latest'
        KUBECONFIG_CREDENTIALS_ID = 'kube-file'
        KUBECONFIG = credentials('kube-file')
        DOCKER_CREDENTIALS_ID = 'docker'
        GIT_CREDENTIALS_ID = 'Aylanesa7'
        GIT_CREDENTIALS = credentials('Aylanesa7')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/ali-miyan/automated-ecommerce-pipeline-with-jenkins.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.REGISTRY}/${env.IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", "${env.DOCKER_CREDENTIALS_ID}") {
                        def dok =  docker.image("${env.REGISTRY}/${env.IMAGE}")
                        dok.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: env.KUBECONFIG_CREDENTIALS_ID]) {
                    sh '''
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }

        stage('Apply Kubernetes Service') {
            steps {
                withKubeConfig([credentialsId: env.KUBECONFIG_CREDENTIALS_ID]) {
                    sh '''
                    kubectl apply -f service.yaml
                    '''
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
