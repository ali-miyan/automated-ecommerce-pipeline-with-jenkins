pipeline {
    agent any

    environment {
        REGISTRY = 'hub.docker.com'
        REPO = 'alimiyn/ecommerce'
        IMAGE = 'alimiyn/ecommerce:latest'
        KUBECONFIG_CREDENTIALS_ID = 'kube-file'
        KUBECONFIG = credentials('kube-file')
        DOCKER_CREDENTIALS_ID = 'docker-details'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    sh '''
                    git config --global http.version HTTP/1.1
                    git config --global http.postBuffer 524288000
                    '''
                }
                retry(3) {
                    script {
                        def branch = 'main'
                        echo "Checking out branch: ${branch}"
                        git url: 'https://github.com/ali-miyan/automated-ecommerce-pipeline-with-jenkins'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${env.REPO}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', env.DOCKER_CREDENTIALS_ID) {
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
