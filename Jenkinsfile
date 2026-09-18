pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t workshop-app:latest .'
            }
        }
        stage('Push') {
            steps {
                sh 'docker tag workshop-app:latest registry.local/workshop-app:latest'
                sh 'docker push registry.local/workshop-app:latest'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}

