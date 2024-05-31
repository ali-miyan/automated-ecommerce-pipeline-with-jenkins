pipeline {
    agent any

    environment {
        REGISTRY = 'hub.docker.com'
        REPO = 'alimiyn/ecommerce'
        IMAGE = 'alimiyn/ecommerce:latest'
        KUBECONFIG_CREDENTIALS_ID = 'kube-file'
        KUBECONFIG = credentials('kube-file')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ali-miyan/ecommerce'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${env.REGISTRY}/${env.REPO}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-details') {
                        dockerImage.push()
                        dockerImage.push('latest')
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
    }

    post {
        always {
            cleanWs()
        }
    }
}
