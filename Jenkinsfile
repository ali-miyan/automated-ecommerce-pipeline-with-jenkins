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
                    withCredentials([usernamePassword(credentialsId: 'Aylanesa7', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {

                        sh '''
                        echo $DOCKER_PASSWORD | docker login registry.hub.docker.com --username $DOCKER_USERNAME --password-stdin
                        '''
                        
                        sh '''
                        docker tag ${IMAGE} registry.hub.docker.com/${IMAGE}
                        docker push registry.hub.docker.com/${IMAGE}
                        '''
                        
                        sh "docker logout registry.hub.docker.com"
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
