pipeline {
    agent any

    environment {
        REGISTRY = 'hub.docker.com'
        REPO = 'alimiyn/ecommerce'
        IMAGE = 'alimiyn/ecommerce:latest'
        KUBECONFIG_CREDENTIALS_ID = 'kube-file'
        KUBECONFIG = credentials('kube-file')
        DOCKER_CREDENTIALS_ID = 'docker-details'
        GIT_CREDENTIALS_ID = 'Aylanesa7'
        GIT_CREDENTIALS = credentials(GIT_CREDENTIALS_ID)
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        git url: 'https://github.com/ali-miyan/automated-ecommerce-pipeline-with-jenkins',
                            credentialsId: GIT_CREDENTIALS_ID,
                            username: USERNAME,
                            password: PASSWORD
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
