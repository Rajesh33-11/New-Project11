# Git → Maven → Jenkins → Docker → DockerHub → K8S Cluster
------------------------------------------------
<img width="1024" height="610" alt="image" src="https://github.com/user-attachments/assets/e5484acd-5334-43ef-bd36-e2fa9fe19bae" />

-------------------------------------------------
## Rerequirements:
#### -CREATE EC2 USING UBUNTU WITH 30 GB EBS AND INSTANCE_TYPE BE M7I-FLUX.LARGE(Eg: 8 CPUS, 32GB RAM),
#### -CREATE IAM ROLE WITH ADMIN ACCESS ADD THE ROLE TO YOUR  EC2 SERVER
------------------------------------------------
# Setup Jenkins
```
vim jenkins.sh
```
```
sudo apt update -y
sudo apt upgrade -y
sudo apt install git openjdk-8-jdk maven -y
sudo apt install -y openjdk-17-jdk
java -version
sudo rm -f /etc/apt/sources.list.d/jenkins.list
sudo rm -f /usr/share/keyrings/jenkins-keyring.*
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y  # Install Java first[page:1]
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
update-alternatives --config java
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
<img width="1886" height="477" alt="image" src="https://github.com/user-attachments/assets/2e54e7b0-00a8-4355-b2ff-cf02b4d9e9d0" />

```
sh jenkins.sh
```
------------------------------
# Install Docker

```
apt install docker.io -y
```
<img width="1603" height="598" alt="image" src="https://github.com/user-attachments/assets/3aebae5e-e694-41b1-937d-18830e30d134" />


------------------------------
# Setup a cluster
### Installing kubectl
```
# Download the latest kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Validate the binary (optional but recommended)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
```
<img width="1422" height="597" alt="image" src="https://github.com/user-attachments/assets/eb3567d8-e1db-4e3c-a3c2-7ced476b5a97" />


### Installing kops
```
# Download the latest kops binary
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

# Make it executable
chmod +x kops-linux-amd64

# Move to system path
sudo mv kops-linux-amd64 /usr/local/bin/kops

# Verify installation
kops version
```
<img width="1910" height="492" alt="image" src="https://github.com/user-attachments/assets/7aa268a5-9371-4b3a-83bb-8ea541206594" />


### Installing AWS CLI
```
apt update && apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
<img width="1040" height="268" alt="image" src="https://github.com/user-attachments/assets/7c795148-fcae-4016-a03a-a85de0e0b667" />


------------------------------
# Create S3 bucket for kOps state store
```
aws s3 mb s3://55rajesh.k8s.locals
aws s3api put-bucket-versioning --bucket 55rajesh.k8s.locals --region ap-south-1 --versioning-configuration Status=Enabled
export KOPS_STATE_STORE=s3://55rajesh.k8s.locals
```
<img width="1437" height="172" alt="image" src="https://github.com/user-attachments/assets/06b173c7-06b9-428c-9681-e3380f5ca627" />


--------------------------------
# This command creates a Kubernetes cluster configuration using kOps on AWS
```
kops create cluster --name=rajesh33.k8s.local --zones=ap-northeast-3a --control-plane-size=m7i-flex.large --control-plane-count=1 --node-count=2 --node-size=t3.micro --image=ami-0ef44b9f9f20f3e57
```
<img width="1892" height="516" alt="image" src="https://github.com/user-attachments/assets/8836d207-cbe5-425a-995d-dde76c136531" />

# update cluster
```
kops update cluster --name rajesh33.k8s.local --yes --admin
```
-------------------------------

--------------------------------
<img width="1912" height="980" alt="image" src="https://github.com/user-attachments/assets/87d5f46b-26e9-4940-9139-8d3878884425" />

# Create a pipeline Clone the website files from GitHub to the server.
```
pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "raja3333"
        IMAGE_NAME = "carrer"
        DOCKERHUB_CREDENTIALS = "dockerhub-creds"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajesh33-11/springboot-mongo-docker.git'
            }
        }

        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                  docker build -t $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER .
                  docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                sh '''
                  sed -i "s/IMAGE_TAG/${BUILD_NUMBER}/g" deployment.yml
                  kubectl apply -f deployment.yml
                '''
            }
        }
    }
}

```
Now install Stage View Plugin
<img width="1516" height="360" alt="image" src="https://github.com/user-attachments/assets/0f34e814-0a79-48b7-bf17-e627e524aabc" />
Now Enter credentials Global Level in Jenkins
<img width="1901" height="907" alt="image" src="https://github.com/user-attachments/assets/05c5a5ac-f7f2-4bb0-8808-51c6f2208770" />

-------------------------
Now give permissions in Server
```
chmod 777 /var/run/docker.sock
```
**1️⃣ Created kube config directory for Jenkins**
```
sudo mkdir -p /var/lib/jenkins/.kube
```

**2️⃣ Copied kubeconfig**
```
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
```

**3️⃣ Changed ownership**
```
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

**4️⃣ Restarted Jenkins**
```
sudo systemctl restart jenkins
```

**5️⃣ Verified as Jenkins user**
```
sudo -u jenkins kubectl get nodes
```

**Output shows:**
STATUS: Ready

