# Git → Maven → Jenkins → Docker → DockerHub → K8S Cluster
------------------------------------------------
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
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install -y jenkins
update-alternatives --config java
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
<img width="1107" height="545" alt="image" src="https://github.com/user-attachments/assets/63154375-4839-4237-b434-439ae904988d" />

```
sh jenkins.sh
```
------------------------------
# Install Docker

```
apt install docker.io -y
```
<img width="1565" height="519" alt="image" src="https://github.com/user-attachments/assets/a66f9c6b-cb7f-4c55-9581-af633e8b0d1d" />

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
<img width="1508" height="570" alt="image" src="https://github.com/user-attachments/assets/ac8bf054-2c2d-461d-9e82-7a3a606c99f4" />

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
<img width="1862" height="465" alt="image" src="https://github.com/user-attachments/assets/0e4eaac3-f207-4155-9785-8119dee24dbb" />

### Installing AWS CLI
```
apt update && apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
<img width="925" height="187" alt="image" src="https://github.com/user-attachments/assets/a0165246-d423-443c-a899-40284fe96b02" />

------------------------------
# Create S3 bucket for kOps state store
```
aws s3 mb s3://55rajesh.k8s.locals
aws s3api put-bucket-versioning --bucket 55rajesh.k8s.locals --region ap-south-1 --versioning-configuration Status=Enabled
export KOPS_STATE_STORE=s3://55rajesh.k8s.locals
```
<img width="1486" height="201" alt="image" src="https://github.com/user-attachments/assets/a2e9aa0d-9582-4dd1-b936-be6f88109ef5" />

--------------------------------
# This command creates a Kubernetes cluster configuration using kOps on AWS
```
kops create cluster --name=rajesh33.k8s.local --zones=ap-northeast-3a --control-plane-size=m7i-flex.large --control-plane-count=1 --node-count=2 --node-size=t3.micro --image=ami-06571d6ae17e327ff
```
<img width="1867" height="537" alt="image" src="https://github.com/user-attachments/assets/c96af9e9-b7c2-45bf-b633-abed61ed36aa" />

# update cluster
```
kops update cluster --name rajesh33.k8s.local --yes --admin
```
--------------------------------
#  create ECR
**go to AWS console → ECR → Click on create → enter name (eg: rajeshtest) → click on create**
<img width="1912" height="772" alt="image" src="https://github.com/user-attachments/assets/3011f9d9-6ddd-48ff-8999-7436aa61d39f" />

-----------------------------------
# I want create Sonarcube container
```
docker run -itd --name  con1 -p 9000:9000 sonarqube:8.7-community
```
<img width="1762" height="450" alt="image" src="https://github.com/user-attachments/assets/68879062-7e3c-4242-9c19-092f9618ee39" />

# Access Sonarcube container
```
Public_ip:9000
```
<img width="1908" height="998" alt="image" src="https://github.com/user-attachments/assets/2edc4416-2bd0-46a5-86c8-1d6d67500542" />
Defualt - User_name: admin, Password: admin
<img width="1918" height="967" alt="image" src="https://github.com/user-attachments/assets/b1eb5ab9-a29a-421f-9ab0-2b4aefc1844b" />

--------------------------------
# Create a pipeline Clone the website files from GitHub to the server.
```
pipeline {
    agent any

    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajesh33-11/char-webapp33.git'
            }
        }
    }
}

```
Now build and verify in Server for files
<img width="1913" height="1002" alt="image" src="https://github.com/user-attachments/assets/4356ab96-fd6a-4d19-a2da-ea76ab8d2bc6" />
<img width="1120" height="171" alt="image" src="https://github.com/user-attachments/assets/ba4f9bbd-2c92-4027-adca-a3946aeb2bb4" />
# Now integrate SonarQube
install SonarQube Scanner plugin in jenkins
<img width="1911" height="557" alt="image" src="https://github.com/user-attachments/assets/6bf0098c-1fab-4982-ab56-544be91ca603" />
### Passes SonarQube creditanials in Jenkins
**Click on New Project**
<img width="1915" height="922" alt="image" src="https://github.com/user-attachments/assets/954da042-f3dc-480b-9b3d-116e07769ed3" />
**Click on Manually**
<img width="1917" height="896" alt="image" src="https://github.com/user-attachments/assets/b6e39bdf-66f8-4e6a-a37a-76845b53ea3d" />
****Enter Project name (eg: maven project)** and click on setup**
<img width="1902" height="725" alt="image" src="https://github.com/user-attachments/assets/26e6416e-36e0-4134-971a-9b78b47dbb5a" />
**Enter name and genarate tocken**
<img width="1852" height="841" alt="image" src="https://github.com/user-attachments/assets/1765516e-f884-4fd8-826c-5fa61531f24b" />
**Copy the Tocken and go to jenikins → creditanials → click on global → Click on adding some creditanials**
<img width="1892" height="767" alt="image" src="https://github.com/user-attachments/assets/f039d598-2c3e-41dc-99cd-c722150b9860" />
<img width="1917" height="975" alt="image" src="https://github.com/user-attachments/assets/be5e4932-f9ff-4518-af3f-cc39a4ef30f0" />
**Now enable Sonar Qube in Jenikins**
Manage Jenikins → system → SonarQube Servers
<img width="1665" height="366" alt="image" src="https://github.com/user-attachments/assets/7ff596f2-0c5c-4168-a7c8-4503da45690d" />
**Click on Add sonar Qube**
Enter name , SonarQube_URl and select Auth_token, click on SAVE
<img width="1712" height="717" alt="image" src="https://github.com/user-attachments/assets/44cdf200-96a8-46d8-9370-35c8f8c18b17" />
**go to SonarQube click on Continue**
<img width="1915" height="768" alt="image" src="https://github.com/user-attachments/assets/61d65583-e79e-4157-9100-96e813b6ded1" />
**Click on Maven and Copy the commands**
<img width="1886" height="930" alt="image" src="https://github.com/user-attachments/assets/77ba964c-7d87-4c76-b2b9-7a661f53c18d" />
**Create a pipeline**
```
pipeline {
    agent any

    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajesh33-11/char-webapp33.git'
            }
        }

        stage("Sonar_Scan") {
            steps {
                sh 'sh sonar.sh'
            }
        }
    }
}

```

<img width="1918" height="906" alt="image" src="https://github.com/user-attachments/assets/eaebccb9-b9ed-4586-a5a1-66bdd2a929b4" />

**Now configure the Maven**
Manage Jenikins → Tools → Maven installations → AddMaven → enter name(eg-my maven) → OK
<img width="1735" height="773" alt="image" src="https://github.com/user-attachments/assets/7293b47c-19c8-4441-a2b3-1da7ad824dfb" />
```
pipeline {
    agent any

    tools {
        maven "mymaven"
    }

    environment {
        AWS_REGION = "us-west-1"
        ECR_REGISTRY = "978163710174.dkr.ecr.us-west-1.amazonaws.com"
        IMAGE_NAME = "rajeshtest"
    }

    stages {

        stage("Git Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajesh33-11/char-webapp33.git'
            }
        }

        stage("Sonar Scan") {
            steps {
                sh 'sh sonar.sh'
            }
        }

        stage("Build with Maven") {
            steps {
                sh 'mvn clean install'
            }
        }

        stage("ECR Login") {
            steps {
                sh '''
                  aws ecr get-login-password --region $AWS_REGION \
                  | docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage("Docker Image Build") {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
                '''
            }
        }

        stage("Docker Tag") {
            steps {
                sh '''
                  docker tag $IMAGE_NAME:${BUILD_NUMBER} \
                  $ECR_REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                '''
            }
        }

        stage("Docker Push") {
            steps {
                sh '''
                  docker push $ECR_REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                '''
            }
        }

        stage("Kubernetes Deployment") {
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
Now Pass Creditionals in Jenikins
```
su - jenkins
```
```
aws configure
```
<img width="1201" height="396" alt="image" src="https://github.com/user-attachments/assets/49e89fec-d0cf-47ca-a901-ba6e718d4fb4" />

```
exit
```
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

