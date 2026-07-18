# Containers on AWS (ECS, EKS, Fargate)

## Amazon ECS (Elastic Container Service)

### Overview
AWS-native container orchestration service.

### Launch Types
- **EC2 Launch Type** — you manage EC2 instances (ECS Agent installed)
- **Fargate Launch Type** — serverless, no infrastructure management

### Key Concepts
- **Cluster** — logical group of tasks/services
- **Task Definition** — blueprint (image, CPU, memory, ports, IAM role)
- **Task** — running instance of a task definition
- **Service** — maintains desired number of tasks, integrates with ALB/NLB

### ECS + ALB
- Dynamic port mapping (multiple tasks on same EC2)
- Path-based routing to different services

### IAM Roles
- **EC2 Instance Role** — ECS Agent permissions (pull images, write logs)
- **Task Role** — permissions for the containerized application itself

## Amazon EKS (Elastic Kubernetes Service)

### Overview
Managed Kubernetes on AWS. Use when you need Kubernetes compatibility.

### Key Features
- Managed control plane (AWS manages master nodes)
- Worker nodes: EC2 (managed/self-managed node groups) or Fargate
- Compatible with standard Kubernetes tools (kubectl, Helm)
- EKS Anywhere for on-premises

### When to Choose EKS
- Multi-cloud / hybrid Kubernetes strategy
- Existing Kubernetes expertise
- Need Kubernetes-specific features

## AWS Fargate

### Overview
Serverless compute for containers (works with both ECS and EKS).

### Key Features
- No EC2 instances to manage
- Pay for vCPU and memory per task
- Each task gets its own ENI (own IP)
- Ephemeral storage: 20 GB - 200 GB

## Amazon ECR (Elastic Container Registry)
- Managed Docker image registry
- Private and public repositories
- Integrated with ECS/EKS
- Image scanning for vulnerabilities
- Lifecycle policies for image cleanup

## Exam Tips
- ECS = AWS native, simpler
- EKS = Kubernetes, more complex, multi-cloud
- Fargate = serverless containers (no EC2 management)
- ECS + Fargate is the most common serverless container pattern
- EC2 launch type when you need GPU, specific instance types, or cost optimization
- Task Role ≠ Instance Role (common exam question)

---

Related: [[Lambda]] | [[Domain 4 - Cost-Optimized Architectures/EC2 Pricing]]
#aws #ecs #eks #fargate #containers #domain3
