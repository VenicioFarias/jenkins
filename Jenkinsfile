pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker login -u veniciofarias -p '0223010@Vf'
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