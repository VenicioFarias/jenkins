pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
               sh 'echo "Executando o comando DockerBuild"'
            }
        }

        stage('Push Docker Image') {
            steps {
                 sh 'echo "Executando o comando Push"'
                }
            }
             

        stage('Deploy no Kubernetes') {
          
            steps {
                sh 'echo "Executando o comando Kubectl apply"'
            }
        } 
    }
}
