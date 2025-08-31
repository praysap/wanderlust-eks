# 🚀 End-to-End MERN Application Deployment on Amazon EKS

This repository contains a complete Kubernetes deployment setup for a MERN (MongoDB, Express.js, React, Node.js) application using:

## 🛠️ Key Features & Tools

- 🧠 **EKS** → Kubernetes cluster provisioning and management  
- 📦 **Helm** → Templated, reusable Kubernetes manifests for simplified deployment  
- 🐳 **Docker** → Containerization of backend, frontend, and database services  
- ⚙️ **Jenkins** (optional) → CI/CD pipeline automation for seamless deployment  
- 🌍 **Terraform & AWS CLI** → Infrastructure as Code (IaC) for AWS resource provisioning  
- 🔒 **ECR** → Secure container image registry for application images
- 📊 **Prometheus** → Metrics collection and alerting for application & cluster monitoring  
- 📈 **Grafana** → Interactive dashboards for real-time visualization and insights   




### Step 1: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region `us-west-2`).
- SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 4: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 5: Build Docker images
``` shell
docker build -t wanderlust-backend
```
 **After the build completes, tag your image so you can push the image to this repository:**
``` shell
docker tag wanderlust-backend:latest public.ecr.aws/x4m1c1q0/wanderlust-backend:latest
```
**Run the following command to push this image to your newly created AWS repository**
``` shell
docker push public.ecr.aws/x4m1c1q0/wanderlust-backend:latest
```

### Step 6: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 7: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 8: Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```
<img width="941" height="459" alt="image (2)" src="https://github.com/user-attachments/assets/4570952b-5cf5-472b-8dcf-9d1e403bb426" />

<img width="800" height="166" alt="image (3)" src="https://github.com/user-attachments/assets/77124c0d-6501-41b4-81e8-159ce50fb7bc" />


### Step 9: Run Manifests
``` shell
kubectl create namespace three-tier
kubectl apply -f .
kubectl delete -f .
```
<img width="1055" height="357" alt="image" src="https://github.com/user-attachments/assets/9c3c4d0d-41ff-4fe0-ab2e-b3287231d673" />
## ⚙️ Jenkins configuration

---

## ✅ Prerequisites

- 🐧 Ubuntu 24.04 system
- 🔑 `sudo` privileges
- ☕ Java (OpenJDK 11 or 17 — recommended: 17)

---

## 🔄 Post-Installation Recommendations

✅ Install the following plugins during setup:

- Pipeline
- Docker Pipeline
- Git
- SSH Agent
- Credentials Binding
- Kubernetes CLI
- Helm

---

## 🛠️ Installation Steps

### 1. 🔄 Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. ☕ Install Java (OpenJDK 17)

```bash
sudo apt install -y openjdk-17-jdk
```

🔍 Verify Java installation:

```bash
java -version
```

### 3. 📦 Add Jenkins Repository and Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### 4. 🧰 Install Jenkins

```bash
sudo apt update
sudo apt install -y jenkins
```

### 5. 🚀 Start and Enable Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check Jenkins service status:

```bash
sudo systemctl status jenkins
```

### 6. 🔥 Open Firewall (if enabled)

```bash
sudo ufw allow 8080
sudo ufw allow OpenSSH
sudo ufw enable
```

### 7. 🌐 Access Jenkins

Open your browser and go to: `http://your_server_ip:8080`

🔑 Retrieve the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Follow the setup wizard to:

- Create your first admin user 👤
- Install recommended plugins 🔌
- Configure Jenkins as needed ⚙️

### 🔧 Optional: Change Jenkins Port

Edit the Jenkins config file:

```bash
sudo nano /etc/default/jenkins
```

Change the line:

```
HTTP_PORT=8080
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

---

### Pipeline setup
1. Create Jenkinsfile inside your project directory.
2. Create ECR credentials in jenkins.
3. Create jenkins pipeline.

```
pipeline {
  agent any
  options {
    timestamps()
  }
  environment {
    AWS_REGION        = 'us-west-2'
    CLUSTER_NAME      = 'three-tier-cluster'
    NAMESPACE         = 'three-tier'
    ECR_REPO_FRONTEND = 'wanderlust-frontend'
    ECR_REPO_BACKEND  = 'wanderlust-backend'
    ECR_REGISTRY      = "863541429677.dkr.ecr.us-west-2.amazonaws.com" // <-- replace with your AWS account ID
  }
  stages {
    stage('Init AWS + Vars') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            echo "AWS credentials and region set"
          }
        }
      }
    }
    stage('Checkout Code') {
      steps {
        git branch: 'devops', url: 'https://github.com/vipulsaw/Capstone-EKS-Application-Deployment-with-Monitoring.git'
      }
    }
    stage('Configure kubeconfig') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              set -e
              aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
            """
          }
        }
      }
    }
    stage('Ensure ECR repos') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              set -e
              aws ecr describe-repositories --repository-names ${ECR_REPO_FRONTEND} >/dev/null 2>&1 || \
                aws ecr create-repository --repository-name ${ECR_REPO_FRONTEND}
              aws ecr describe-repositories --repository-names ${ECR_REPO_BACKEND} >/dev/null 2>&1 || \
                aws ecr create-repository --repository-name ${ECR_REPO_BACKEND}
            """
          }
        }
      }
    }
    stage('Login to ECR') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              aws ecr get-login-password --region ${AWS_REGION} | \
              docker login --username AWS --password-stdin ${ECR_REGISTRY}
            """
          }
        }
      }
    }
    stage('Build Frontend Image') {
      steps {
        sh """
          cd frontend
          docker build -t ${ECR_REGISTRY}/${ECR_REPO_FRONTEND}:latest .
        """
      }
    }
    stage('Build Backend Image') {
      steps {
        sh """
          cd backend
          docker build -t ${ECR_REGISTRY}/${ECR_REPO_BACKEND}:latest .
        """
      }
    }
    stage('Push Images to ECR') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              docker push ${ECR_REGISTRY}/${ECR_REPO_FRONTEND}:latest
              docker push ${ECR_REGISTRY}/${ECR_REPO_BACKEND}:latest
            """
          }
        }
      }
    }
    stage('Apply Manifests to EKS') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              echo "Applying backend deployment and service..."
              kubectl apply -f k8s-Manifests/backend/deployment.yaml -n ${NAMESPACE}
              kubectl apply -f k8s-Manifests/backend/service.yaml -n ${NAMESPACE}
              echo "Waiting for backend pods to be ready..."
              kubectl rollout status deployment/api -n ${NAMESPACE} --timeout=180s
              echo "Applying frontend deployment and service..."
              kubectl apply -f k8s-Manifests/frontend/deployment.yaml -n ${NAMESPACE}
              kubectl apply -f k8s-Manifests/frontend/service.yaml -n ${NAMESPACE}
              echo "Waiting for frontend pods to be ready..."
              kubectl rollout status deployment/frontend -n ${NAMESPACE} --timeout=180s
            """
          }
        }
      }
    }
  }
}
```

<img width="1079" height="652" alt="image" src="https://github.com/user-attachments/assets/fcb50ad1-c83d-4d47-a3e8-4089b0627b87" />

<img width="1080" height="661" alt="wanderlust-landing" src="https://github.com/user-attachments/assets/1680dbec-9101-44db-b2b3-3bb369d92482" />

<img width="1080" height="662" alt="Create_Blog" src="https://github.com/user-attachments/assets/6abf4b60-49cd-40ab-b97b-c16324088da0" />

<img width="1079" height="665" alt="blog-added" src="https://github.com/user-attachments/assets/33676805-82f3-466e-8e8a-87ad293097d5" />

<img width="955" height="500" alt="image" src="https://github.com/user-attachments/assets/49708fe4-f474-45f8-bb06-9c1e74e05c50" />

<img width="957" height="497" alt="image (1)" src="https://github.com/user-attachments/assets/5a89a16d-60ef-4c5c-8721-555d1a39f110" />

### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```
- To clean up rest of the stuff and not incure any cost
```
Stop or Terminate the EC2 instance created in step 2.
Delete the Load Balancer created in step 9 and 10.
Go to EC2 console, access security group section and delete security groups created in previous steps
```




---
Happy Learning! 🚀👨‍💻👩‍💻
