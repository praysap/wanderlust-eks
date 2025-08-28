# 🚀 MERN Application Deployment on Kubernetes (EKS)

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



## Application Code
The `Application-Code` directory contains the source code for the Three-Tier Web Application. Dive into this directory to explore the frontend and backend implementations.

## Jenkins Pipeline Code
In the `Jenkins-Pipeline-Code` directory, you'll find Jenkins pipeline scripts. These scripts automate the CI/CD process, ensuring smooth integration and deployment of your application.

## Jenkins Server Terraform
Explore the `Jenkins-Server-TF` directory to find Terraform scripts for setting up the Jenkins Server on AWS. These scripts simplify the infrastructure provisioning process.

## Kubernetes Manifests Files
The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on AWS EKS. Understand and customize these files to suit your project needs.


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
