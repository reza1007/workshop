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
4. Create Multi‑Node Kubernetes Cluster
By default, Kind creates a single‑node cluster. For this workshop, we’ll use 1 control plane + 1 worker node.

Delete any existing cluster:

bash
kind delete cluster --name workshop
Create a config file kind-cluster.yaml:

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
Create the new cluster:

bash
kind create cluster --name workshop --config kind-cluster.yaml
Verify nodes:

bash
kubectl get nodes
Expected:

Code
workshop-control-plane   Ready   control-plane
workshop-worker          Ready   <none>

# Label the worker node
kubectl label nodes kind-worker node-role.kubernetes.io/worker=""
5. Install Java 21 + Jenkins
bash
sudo apt install openjdk-21-jdk -y
java -version

Option A: Install via Package Repo
bash
sudo apt install openjdk-21-jdk -y
java -version

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins=2.568.3 -y
Option B: Manual .deb Install
bash
wget https://pkg.jenkins.io/debian-stable/binary/jenkins_2.568.3_all.deb
sudo apt install ./jenkins_2.568.3_all.deb -y
Enable and start Jenkins:

bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
Access Jenkins: http://localhost:8080  
Unlock Jenkins:

bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Configure kubeconfig for Jenkins
By default, Jenkins runs as the jenkins user and cannot access your personal kubeconfig. Copy it into Jenkins’s home:

bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
Verify:

bash
sudo -u jenkins kubectl get nodes
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
kubectl port-forward svc/workshop-app-service 8081:80
Open in browser: http://localhost:8081

# if use ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml\n
sudo vi ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: workshop-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: workshop-app-service
            port:
              number: 80

kubectl apply -f ingress.yaml\n
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8082:80
Open in browser: http://localhost:8082/app

📊 Architecture Diagram
Code
GitHub Repo  --->  Jenkins Pipeline  --->  DockerHub (Image Registry)  --->  Kubernetes Cluster  --->  Web Browser (App)
✅ End Result:

WSL2 + Ubuntu installed

Docker, Kind, kubectl set up

Multi‑node cluster (1 control plane + 1 worker) created

Jenkins installed with Java 21

GitHub repo connected

DockerHub credentials configured

Pipeline builds → pushes image to DockerHub → deploys to Kubernetes

App accessible in browser at http://localhost:8082/app
