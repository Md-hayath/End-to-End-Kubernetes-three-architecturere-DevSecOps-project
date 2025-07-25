# End-to-End-Kubernetes-three-architecturere-DevSecOps-project
Three-Tier Web Application Deployment on AWS EKS using AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins

![Three Tier Architecture](https://github.com/Md-hayath/yml/blob/82ad248468aeb229b37392aa0eb39f3b5f14b5c1/Three-Tier.gif?raw=true)


  PART A: Jenkins EC2 Setup (t2.xlarge)
1. Create an EC2 instance with the following specs:

Instance type: t2.xlarge

OS: Ubuntu 20.04 (recommended)

Volume: At least 50 GB

Open ports: 22, 9090, 8000

Assign an IAM role with permissions: EC2, S3, EKS, IAM access.

2. Before launching the instance, use a User Data script to install:

Terraform

Docker

JDK

Jenkins

SonarQube

Trivy

3. Launch the EC2 instance.

4. SSH into the EC2 instance:

bash
Copy
Edit
ssh -i <your-key.pem> ubuntu@<EC2-IP>
5. Run the following to verify:

bash
Copy
Edit
sudo java --version  # Check JDK
docker --version     # Docker check
jenkins --version    # Optional, if CLI installed
terraform version    # Terraform
trivy --version      # Trivy
🔧 PART B: Jenkins Initial Configuration
6. Access Jenkins on: http://<EC2-IP>:9090

7. Create a Jenkins user (instead of using admin) during setup.

8. Install required Jenkins plugins:

AWS Credentials

AWS CodePipeline

Pipeline

Stage View

9. Go to:

Manage Jenkins → Manage Plugins → Install plugins above if not already.

  PART C: Jenkins IAM Credential Setup
10. Navigate to:

Manage Jenkins → Credentials → (global) → Add Credentials

11. Use IAM credentials:

Go to AWS Console → IAM → Users → Security Credentials → Create access key

Copy Access Key ID and Secret Key

12. Back in Jenkins, choose:

Type: AWS Credentials

Scope: Global

Paste Access & Secret Keys

ID: aws-creds (for reference in pipeline)

  PART D: Terraform Plugin and Tools Configuration
13. Go to:

Manage Jenkins → Global Tool Configuration

14. Scroll to Terraform section (may not appear until plugin is installed).

If not visible, install the Terraform Plugin via Plugin Manager.

15. Configure Terraform path manually:

bash
Copy
Edit
whereis terraform
# Example output: /usr/bin/terraform
Paste this path in the Terraform installation section in Jenkins.

  PART E: Jenkinsfile Pipeline (with Terraform)
16. Clone or fetch the repo that contains:

Terraform code (main.tf)

Jenkinsfile pipeline script

17. In Jenkins:

Create a new Pipeline project

In "Pipeline Script", paste your Jenkinsfile content

18. Understand your Jenkinsfile stages:

terraform init – initializes backend

terraform validate – checks syntax

terraform plan – shows changes

terraform apply – applies infra changes

Make sure conditional logic exists for choice parameters if needed

 PART F: Jenkins Stage View
19. Install Stage View plugin (if not already)

20. After building a pipeline, go to the project → Stage View tab

You’ll see visual breakdown of all stages: Init → Plan → Apply, etc.

 PART G: Jump Server EC2 Setup (t2.medium)
21. Create a second EC2 instance:

Type: t2.medium

Volume: 30 GB

OS: Ubuntu

IAM role: Same or similar to Jenkins (for AWS & EKS access)

22. SSH into the jump server:

bash
Copy
Edit
ssh -i <your-key.pem> ubuntu@<jump-server-IP>
23. Install required tools:

bash
Copy
Edit
sudo apt update && sudo apt install -y awscli kubectl helm
24. Install EKS CLI:

bash
Copy
Edit
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
25. Run aws configure:

Use the same IAM credentials added to Jenkins.

 PART H: Wait for EKS Cluster Creation via Jenkins
26. After running Jenkins pipeline that provisions EKS (if defined in Terraform):

Wait until cluster is created successfully

27. From Jump Server, verify access:

bash
Copy
Edit
aws eks --region <region> update-kubeconfig --name <cluster_name>
kubectl get nodes
