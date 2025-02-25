## Overview

Deployinga **Three-Tier Web Application** on **AWS EKS**.

## Prerequisites

Before starting, ensure you have:

- Basic knowledge of **Docker** and **AWS services**.
- An **AWS account** with necessary permissions.

### Project Details

### Application Code

The `Application-Code` directory contains the **frontend and backend** source code for the Three-Tier Web Application.

### Kubernetes Manifests Files

The `Kubernetes-Manifests-Files` directory contains **Kubernetes manifests** to deploy the application on **AWS EKS**.

## Getting Started

Follow these steps to set up and deploy the application:

### Step 1: IAM Configuration

1. Create an **IAM User** `eks-admin` with `AdministratorAccess`.
2. Generate **Access Key** and **Secret Access Key**.

### Step 2: EC2 Setup

1. Launch an **Ubuntu instance** in your preferred AWS region.
2. SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2

```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 4: Install Docker

```sh
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 5: Install kubectl

```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 6: Install eksctl

```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 7: Setup EKS Cluster

```sh
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

### Step 8: Run Kubernetes Manifests

```sh
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```

### Step 9: Install AWS Load Balancer Controller

```sh
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

### Step 10: Deploy AWS Load Balancer Controller

```sh
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```

## Cleanup

To delete the EKS cluster and other resources:

```sh
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

To avoid unnecessary AWS costs:

- Stop or terminate the **EC2 instance** created.
- Delete the **Load Balancer** created.
- Go to the **EC2 console** and delete unnecessary **security groups**.

## Contribution Guidelines

1. **Fork** the repository and create a **feature branch**.
2. Deploy the application and add **creative enhancements**.
3. Ensure your code follows the **project's style and contribution guidelines**.
4. Submit a **Pull Request (PR)** with a detailed description of your changes.

---
