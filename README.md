# CD - Deploy to EKS Cluster from Jenkins Pipeline

This project focuses on setting up a Continuous Deployment (CD) pipeline to deploy applications to an AWS EKS cluster using Jenkins.

## Technologies:
- Kubernetes
- Jenkins
- AWS EKS
- Docker
- Linux

## Project Description
1. Install `kubectl` and `aws-iam-authenticator` on a Jenkins server.
2. Create a `kubeconfig` file to connect to the EKS cluster and add it to the Jenkins server.
3. Add AWS credentials to Jenkins for account authentication.
4. Modify the Jenkinsfile from the previous CI/CD pipeline to configure the connection to the EKS cluster.

---

## Step-by-Step Guide

### 1. Install `kubectl` Inside Jenkins Container
#### Steps:
1. SSH into the instance hosting the Jenkins container.
2. List running Docker containers to identify the Jenkins container:
   ```bash
   docker ps
   ```
3. Enter the Jenkins container as a root user:
   ```bash
   docker exec -u 0 -it <container-ID> bash
   ```
4. Install `kubectl`:
   ```bash
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mv ./kubectl /usr/local/bin/kubectl
   ```
5. Verify the installation:
   ```bash
   kubectl version
   ```

### 2. Install `aws-iam-authenticator` Inside Jenkins Container
#### Steps:
1. Download the `aws-iam-authenticator` binary:
   ```bash
   curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
   chmod +x ./aws-iam-authenticator
   mv ./aws-iam-authenticator /usr/local/bin
   ```
2. Verify the installation:
   ```bash
   aws-iam-authenticator help
   ```
3. Exit the container:
   ```bash
   exit
   ```

### 3. Create a `kubeconfig` File for EKS Cluster
#### Steps:
1. Update the following details in the `kubeconfig` template:
   - Cluster name: `demo-cluster`
   - API server endpoint: From the EKS cluster overview in AWS.
   - Certificate-authority-data: Copy from the local `~/.kube/config` file.

#### `kubeconfig` File Template:
```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <certificate-authority-data>
    server: https://03E29CF6294E6A6033372CDF0242CEB7.gr7.us-east-1.eks.amazonaws.com
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: /usr/local/bin/aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "demo-cluster"
```

2. Save the file as `config`.
3. Copy the `config` file to the Jenkins container:
   ```bash
   docker cp config <container-ID>:/var/jenkins_home/.kube/
   ```

### 4. Add AWS Credentials to Jenkins
#### Steps:
1. Open the Jenkins dashboard and configure the project as a multi-branch pipeline.
2. Navigate to **Manage Jenkins > Credentials** and add the following credentials:
   - **AWS Access Key ID:**
     - Type: Secret text
     - ID: `jenkins-aws_access_key_id`
     - Secret: Value of the `aws_access_key_id`
   - **AWS Secret Access Key:**
     - Type: Secret text
     - ID: `jenkins-aws_secret_access_key`
     - Secret: Value of the `aws_secret_access_key`

### 5. Adjust Jenkinsfile to Configure EKS Deployment
#### Sample Jenkinsfile:
```groovy
pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Build Application') {
            steps {
                script {
                    echo 'Building the application...'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building the Docker image..."
                }
            }
        }
        stage('Deploy to EKS') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins-aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
            }
            steps {
                script {
                    echo 'Deploying the application to EKS...'
                    sh 'kubectl create deployment nginx-deployment --image nginx'
                }
            }
        }
    }
}
```
Next, execute the project's Jenkins pipeline and once the run is successfully completed, check the pods running: 
```bash
kubectl get pods
```

---


