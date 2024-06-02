pipeline {
    agent any

    environment {
        REGISTRY = 'hub.docker.com'
        REPO = 'alimiyn/ecommerce'
        IMAGE = 'alimiyn/ecommerce'
        TAG = 'latest'
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
                    docker.build("${env.IMAGE}:${env.TAG}")
                }
            }
        }   

        stage('Verify Docker Image') {
            steps {
                sh 'docker images'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {

                        sh '''
                        echo $DOCKER_PASSWORD | docker login registry.hub.docker.com --username $DOCKER_USERNAME --password-stdin
                        '''
                        
                        sh '''
                        docker tag ${IMAGE}:${TAG} registry.hub.docker.com/${IMAGE}:${TAG}
                        docker push registry.hub.docker.com/${IMAGE}:${TAG}
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
