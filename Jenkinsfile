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
        	withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh 'docker tag workshop-app:latest $DOCKER_USER/workshop-app:latest'
            sh 'docker push $DOCKER_USER/workshop-app:latest'
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

