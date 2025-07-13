pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker build -t sathish103/paymentservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker push sathish103/paymentservice:latest "
                    }
                }
            }
        }
    }
}
