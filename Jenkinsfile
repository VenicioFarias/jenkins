pipeline {
    agent any

    stages {
         stage('Docker Login') {
            steps {
                script {
                    // Método 1: Usando withCredentials (RECOMENDADO)
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        sh '''
                            echo "Fazendo login no Docker Hub..."
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        '''
                    }
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    dockerapp = docker.build("veniciofarias/jenkins-guia:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                 script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy no Kubernetes') {
            steps {
                sh 'echo "Executando o comando Kubectl apply"'
            }
        }
    }
}