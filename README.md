markdown
# 🚀 Kubernetes + Jenkins Workshop

This guide walks you through setting up **WSL2 + Ubuntu**, installing **Docker, Kind, kubectl**, and deploying an app to **Kubernetes via Jenkins pipeline with DockerHub integration**.

---

## 1. Enable WSL2 & Install Ubuntu
Open **PowerShell (Admin)**:
```powershell
wsl --install
Restart → open Ubuntu app → update packages:

bash
sudo apt update && sudo apt upgrade -y
2. Install Docker
bash
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
(log out/in to apply group change)

3. Install Kind & kubectl
bash
# Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
4. Create Kubernetes Cluster
bash
kind create cluster --name workshop
kubectl get nodes
5. Install Java 21 + Jenkins
bash
sudo apt install openjdk-21-jdk -y
java -version

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
Access Jenkins: http://localhost:8080  
Unlock Jenkins:

bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
6. Use GitHub Repo
Fork or clone this repo into your own GitHub account:

bash
git clone https://github.com/reza1007/workshop.git
cd workshop
git remote remove origin
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
7. Connect Jenkins to Repo
Create new Pipeline job in Jenkins.

Select Pipeline script from SCM → Git.

Paste your repo URL.

Branch: main.

8. Configure DockerHub Authentication
Create a DockerHub account at https://hub.docker.com.

In Jenkins → Manage Jenkins → Credentials → Global credentials → Add Credentials.

Type: Username with password

Username = DockerHub username

Password = DockerHub password or Access Token

ID = dockerhub-credentials

Save.

9. Jenkinsfile Example
Update your Jenkinsfile to include DockerHub login:

groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps { sh 'docker build -t <dockerhub-username>/workshop-app .' }
    }
    stage('Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
      }
    }
    stage('Push') {
      steps { sh 'docker push <dockerhub-username>/workshop-app:latest' }
    }
    stage('Deploy') {
      steps { sh 'kubectl apply -f k8s/deployment.yaml' }
    }
  }
}
10. Verify & Access App
bash
kubectl get pods
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8082:80
Open in browser: http://localhost:8082/app

✅ End Result:

WSL2 + Ubuntu installed

Docker, Kind, kubectl set up

Kubernetes cluster created

Jenkins installed with Java 21

GitHub repo connected

DockerHub credentials configured

Pipeline builds → pushes image to DockerHub → deploys to Kubernetes

App accessible in browser at http://localhost:8082/app
