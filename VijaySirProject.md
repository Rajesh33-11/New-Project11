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
<img width="1123" height="492" alt="image" src="https://github.com/user-attachments/assets/a9e99ec6-f630-4344-8328-1ee6c5f373a6" />


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
<img width="1472" height="596" alt="image" src="https://github.com/user-attachments/assets/7920c137-243c-4ec9-8092-1bffe67661ce" />



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
<img width="1897" height="450" alt="image" src="https://github.com/user-attachments/assets/c7696798-2a3a-462a-86a0-9535303a0776" />



### Installing AWS CLI
```
apt update && apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
<img width="1167" height="273" alt="image" src="https://github.com/user-attachments/assets/44b47c05-65c0-46c7-b283-0f628e5d8151" />


------------------------------
# Create S3 bucket for kOps state store
```
aws s3 mb s3://55rajesh.k8s.locals
aws s3api put-bucket-versioning --bucket 55rajesh.k8s.locals --region ap-south-1 --versioning-configuration Status=Enabled
export KOPS_STATE_STORE=s3://55rajesh.k8s.locals
```
<img width="1457" height="212" alt="image" src="https://github.com/user-attachments/assets/74f5b301-79db-433c-afd1-10d9ae58a687" />

--------------------------------
# This command creates a Kubernetes cluster configuration using kOps on AWS
```
kops create cluster --name=rajesh44.k8s.local --zones=ap-northeast-2a --control-plane-size=m7i-flex.large --control-plane-count=1 --node-count=2 --node-size=t3.micro --image=ami-0092e0c93f74c293a
```
<img width="1944" height="163" alt="image" src="https://github.com/user-attachments/assets/f45a53a8-f62d-4d3b-be1e-edc56bfbd344" />

# update cluster
```
kops update cluster --name rajesh33.k8s.local --yes --admin
```
<img width="1310" height="252" alt="image" src="https://github.com/user-attachments/assets/ba1219b7-59af-4359-bcbe-ba0ad3a2d2a8" />

-------------------------------
<img width="883" height="121" alt="image" src="https://github.com/user-attachments/assets/f600218d-4ab1-4243-9781-cc8515c2b5ff" />


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
<img width="750" height="213" alt="image" src="https://github.com/user-attachments/assets/d1ee78a3-7563-4039-a60c-57d6c2158e0b" />

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
<img width="1097" height="337" alt="image" src="https://github.com/user-attachments/assets/d8af0d33-dd45-41b1-8386-a4fdb4eb3285" />
<img width="1912" height="982" alt="image" src="https://github.com/user-attachments/assets/6de860a6-9b2e-4304-a15c-2214c7e222a8" />
<img width="1587" height="192" alt="image" src="https://github.com/user-attachments/assets/6f863ad4-dea4-414f-bfec-fdaec934940a" />
<img width="1918" height="912" alt="image" src="https://github.com/user-attachments/assets/da77b2e8-706a-4f2f-935b-30271a636917" />
<img width="1918" height="1008" alt="image" src="https://github.com/user-attachments/assets/c3a83775-f36a-447f-a5c9-0854e145c662" />




