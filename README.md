# What You’ll Do
```bash
Create Ec2-server with ubuntu image (t2.xlarge) 30gb storage, key-pair
Security group - allow ports
8080, 3000, 9090, 9000, 32630, 22, 8081, 6443, 465, 80, 9115
# install this below tools
AWS CLI
kubectl
eksctl
docker
Jenkins
EKS cluster setup (with OIDC & managed node group)
```

# Step-by-Step Shell Script (setup.sh)
```bash
#!/bin/bash

set -e

### ========== AWS CLI v2 ========== ###
echo "👉 Installing AWS CLI v2..."
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt update -y && sudo apt install unzip -y
unzip -o awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/

### ========== kubectl ========== ###
echo "👉 Installing kubectl..."
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

### ========== eksctl ========== ###
echo "👉 Installing eksctl..."
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

### ========== Jenkins ========== ###
echo "👉 Installing Jenkins..."
sudo apt update -y
sudo apt install -y openjdk-17-jre-headless wget curl gnupg2
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
echo "✅ Jenkins Status:"
sudo systemctl status jenkins --no-pager
echo "🔑 Jenkins Admin Password:"
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

### ========== Docker ========== ###
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

### ========== EKS Cluster ========== ###
echo "👉 Creating EKS Cluster (No Node Group)..."
eksctl create cluster \
  --name=EKS-1 \
  --region=us-east-2 \
  --zones=us-east-2a,us-east-2b \
  --without-nodegroup

### ========== OIDC Provider ========== ###
echo "👉 Associating IAM OIDC provider..."
eksctl utils associate-iam-oidc-provider \
  --region us-east-2 \
  --cluster EKS-1 \
  --approve

### ========== Node Group ========== ###
echo "👉 Creating managed node group..."
eksctl create nodegroup \
  --cluster=EKS-1 \
  --region=us-east-2 \
  --name=node2 \
  --node-type=t3.medium \
  --nodes=3 \
  --nodes-min=2 \
  --nodes-max=4 \
  --node-volume-size=20 \
  --ssh-access \
  --ssh-public-key=DevOps \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access

echo "✅ Setup Complete!"

```


# Jenkins plugins install
```bash
 Docker: Enables Jenkins to use Docker containers. 
 Docker Pipeline: Allows Jenkins to use Docker containers in pipeline 
jobs. 
 Kubernetes: Provides support for Kubernetes in Jenkins. 
 Kubernetes CLI: Allows Jenkins to interact with Kubernetes clusters. 
 Multibranch Scan Webhook Trigger: Adds webhook trigger functionality 
for multibranch projects.
 pipeline stage view

```

# Setup the credendtial in Manage jenkins- UI
```bash
# dockerhub creds   (docker-creds)
# github creds with token (github)
# k8s service account (service-account)
```


# Step-by-Step from Start (for your case)

1. Clone the original repo (with all branches)
```bash 
git clone https://github.com/original-user/microservice.git
cd microservice
```
🔹 2. List all remote branches
```bash
git branch -r
```
You’ll see:
origin/main
origin/cartservice
origin/shippingservice
origin/productcatalogservice
...

3. Checkout each remote branch one by one
```bash
git checkout -b cartservice origin/cartservice

#This creates a local branch from the remote.
```


# Clone a public repo with 13 branches
```bash
    Modify each branch locally
    Push all branches to your own new GitHub repo
    Repo name: microservices-project
```
1. Clone the original repo
```bash
git clone https://github.com/jaiswaladi246/Microservice.git
cd Microservice
```

2. View all remote branches
```bash
git branch -r  ----> it will remote branches list
git branch     ----> it will show only Local repo branches list
git branch -a  ----> it will show both remote and local repo branches list
```

3. Create your new GitHub repo
```bash
Go to GitHub
Click ➕ → New Repository
Repo name: microservices-project
Do NOT initialize with README or license (empty repo)
Copy your repo URL (e.g., https://github.com/sathish103/microservices-project.git)
```

4. Add your new GitHub repo as remote
```bash
git remote rename origin source   # rename old remote
git remote add origin https://github.com/sathish103/microservices-project.git

#You now have:
#source → original public repo
#origin → your GitHub repo
```

5. Fetch all original branches safely
```bash
git remote -v
git fetch source
```

6. Checkout, edit, and push each branch (13 total) ---> Now repeat for each branch 

Branch: main    
```bash
git checkout main source/main
git push -u origin main
```

#Branch: shippingservice
```bash
git checkout -b shippingservice source/shippingservice
git push -u origin shippingservice
```
#Branch: Infra-Steps
```bash
git checkout -b Infra-Steps source/Infra-Steps
git push -u origin Infra-Steps
```

#Branch: cartservice
```bash
git checkout -b cartservice source/cartservice
git push -u origin cartservice
```

#Branch: recommendationservice
```bash
git checkout -b recommendationservice source/recommendationservice
git push -u origin recommendationservice
```

#Branch: productcatalogservice
```bash
git checkout -b productcatalogservice source/productcatalogservice
git push -u origin productcatalogservice
```

#Branch: paymentservice
```bash
git checkout -b paymentservice source/paymentservice
git push -u origin paymentservice
```

#Branch: loadgenerator
```bash
git checkout -b loadgenerator source/loadgenerator
git push -u origin loadgenerator
```

#Branch: frontend
```bash
git checkout -b frontend source/frontend
git push -u origin frontend
```

#Branch: emailservice
```bash
git checkout -b emailservice source/emailservice
git push -u origin emailservice
```

#Branch: currencyservice
```bash
git checkout -b currencyservice source/currencyservice
git push -u origin currencyservice
```

#Branch: checkoutservice
```bash
git checkout -b checkoutservice source/checkoutservice
git push -u origin checkoutservice
```

#Branch: adservice
```bash
git checkout -b adservice source/adservice
git push -u origin adservice
```

7. Verify your GitHub repo
```bash
Go to:
https://github.com/sathish103/microservices-project
You will see all 13 branches under the Branches tab.
```
8. Optional: Remove old source remote (safe)
```bash
#After everything is pushed:

git remote remove source
```


# Create the below objects in k8s
# service-account.yaml
```bash

apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps

```
# role.yaml
```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete

```

# role-binding.yaml
```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps

```

# secret.yaml
```bash

apiVersion: v1
kind: Secret
metadata:
  name: mysecretname
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
type: kubernetes.io/service-account-token

#  kubectl describe secret mysecretname -n webapps ---> take this token and create "service-account" creds in Jenkins UI > Manage Jenkins > Credentials

```


# Notes:
```bash
# need to update the "Docker username" in the Jenkinsfile of every microservice branch to reflect the new Docker Hub account (except main branch) ----> make changes in each branch and commit and push with relavant branch one by one. (Use VS Code for better)

# In the main branch need to update both Jenkinsfie with "new creds" and deployment.yaml file with new "docker image".
```

# Once after completion of all above step
```bash
# create one multibranch piepline with github url it will creae create all jobs based on the branches.
```