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
        	script {
            		docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                	docker.image("reza1007/workshop-app:latest").push()
  		       }
       	 }
    }
}

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}

