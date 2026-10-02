# Trend App - DevOps CI/CD Pipeline

An end-to-end DevOps project demonstrating:
- **Containerization** with Docker
- **Infrastructure as Code** with Terraform
- **Orchestration** with Kubernetes (AWS EKS)
- **CI/CD Automation** with Jenkins
- **Monitoring** with Prometheus + Grafana

##  Quick Links

| Resource | URL / Value |
|----------|-------------|
| GitHub Repo | https://github.com/Ashok9556/Trend |
| DockerHub Image | https://hub.docker.com/r/ashok9951/trend-app |
| Jenkins UI | http://34.233.69.143:8080 |
| Application (K8s LoadBalancer) | http://abb5042761f1d4cdfb8b085367706099-7a1cbb2a0094ee87.elb.us-east-1.amazonaws.com |
| Grafana Dashboard | http://a3f0b642d9d184961a6dfb5102dc1b4f-1171254828.us-east-1.elb.amazonaws.com |
| App LoadBalancer ARN | `arn:aws:elasticloadbalancing:us-east-1:401466818325:loadbalancer/net/abb5042761f1d4cdfb8b085367706099/7a1cbb2a0094ee87` |

##  Architecture

    
                         AWS Cloud (us-east-1)                   
                                                                 
         
                VPC 10.0.0.0/16 (trend-devops-vpc)            
                                                              
        Public Subnets        Private Subnets                
                             
         Jenkins EC2          EKS Nodes                  
          (t3.medium)         (2x t3.med)                
          + Docker                                       
          + kubectl           trend-app                  
          + Terraform         pods                       
                             
                                                           
                             
                                                           
                                     
                                EKS Control Plane          
                                (K8s v1.31)                
                                     
         
    

    CI/CD Flow:
    GitHub Push  Webhook  Jenkins  Docker Build  DockerHub Push
                                              
                                       kubectl apply  EKS Cluster
                                              
                                    LoadBalancer (NLB)  User

##  Tech Stack

| Layer | Technology |
|-------|-----------|
| Application | Static site (Nginx serving pre-built dist/) |
| Container | Docker (nginx:alpine base) |
| Registry | DockerHub |
| IaC | Terraform v1.16.2 |
| Cloud | AWS (VPC, IAM, EC2, EKS, NLB) |
| Orchestration | Kubernetes v1.31 on AWS EKS |
| CI/CD | Jenkins (declarative pipeline) |
| VCS | GitHub + Webhooks |
| Monitoring | Prometheus + Grafana (kube-prometheus-stack) |

##  Repository Structure

    .
     dist/                    # Pre-built application files
     k8s/                     # Kubernetes manifests
        deployment.yaml
        service.yaml
     Dockerfile               # Nginx-based image
     .dockerignore
     .gitignore
     Jenkinsfile              # CI/CD pipeline definition
     README.md

##  Setup Instructions

### 1. Local Docker Test

    docker build -t trend-app:v1 .
    docker run -d -p 3000:80 --name trend-app trend-app:v1
    # Visit http://localhost:3000

### 2. Terraform Infrastructure

    cd terraform
    terraform init
    terraform plan
    terraform apply

Creates:
- VPC with 2 public + 2 private subnets
- Internet Gateway + NAT Gateway
- IAM roles for Jenkins, EKS Cluster, EKS Nodes
- Jenkins EC2 (t3.medium) with Docker, kubectl, Terraform, AWS CLI
- EKS Cluster v1.31 with 2 worker nodes (t3.medium)
- OIDC provider for IRSA

### 3. Kubernetes Deployment

    aws eks update-kubeconfig --region us-east-1 --name trend-devops-eks
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    kubectl get svc trend-app-service

### 4. Jenkins Pipeline

The `Jenkinsfile` defines these stages:
1. **Checkout** - Pull code from GitHub
2. **Build Docker Image** - Build and tag with BUILD_NUMBER
3. **Push to DockerHub** - Push versioned + latest tags
4. **Configure kubectl for EKS** - Update kubeconfig
5. **Deploy to Kubernetes** - Apply manifests + wait for rollout

**Triggers:**
- GitHub webhook on push to `main` (auto-build)
- Manual: Jenkins UI  Build Now

##  Credentials

| Credential ID | Type | Purpose |
|---------------|------|---------|
| `dockerhub-creds` | Username/Password | Push to DockerHub |
| `github-creds` | Username/Token | GitHub access |
| AWS access | IAM Instance Profile | Attached to Jenkins EC2 (no static keys) |

##  Monitoring

Prometheus + Grafana installed via Helm:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

Access Grafana:
- URL: http://a3f0b642d9d184961a6dfb5102dc1b4f-1171254828.us-east-1.elb.amazonaws.com
- Username: `admin`
- Password: retrieve via `kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d`

##  Cleanup

To avoid AWS charges, destroy all resources:

    cd terraform
    terraform destroy

    # Also delete the Grafana/App LoadBalancers
    kubectl delete svc trend-app-service -n default
    kubectl delete svc monitoring-grafana -n monitoring



##  Author

**Ashok Kumar**
- GitHub: [@Ashok9556](https://github.com/Ashok9556)
- DockerHub: [ashok9951](https://hub.docker.com/u/ashok9951)

##  Notes

- The `dist/` folder contains the pre-built application  no `package.json` is needed.
- The Jenkins EC2 is in a public subnet and uses an IAM instance profile for AWS access.
- The EKS control plane allows traffic from the Jenkins SG on port 443.
