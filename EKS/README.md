1. What is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is a fully managed Kubernetes service by AWS. It simplifies the process of deploying, managing, and scaling containerized applications using Kubernetes without having to manage the Kubernetes control plane.

2. What is Kubernetes?

Kubernetes is an open-source platform for automating the deployment, scaling, and management of containerized applications. It organizes containers into pods, manages their lifecycle, and ensures high availability.

3. Why should I use Amazon EKS?

Managed Kubernetes Control Plane: AWS handles upgrades, patches, and high availability.

Integration with AWS Services: EKS integrates with IAM, CloudWatch, and more.

Security: Supports IAM for RBAC and VPC isolation.

Scalability: Easily scale workloads using Kubernetes-native tools or AWS integrations.

4. How does Amazon EKS work?

Amazon EKS runs the Kubernetes control plane across multiple Availability Zones to ensure high availability. The worker nodes can run on Amazon EC2, AWS Fargate, or on-premises infrastructure using EKS Anywhere.

Key Components:

Control Plane: Managed by AWS.

Worker Nodes: Managed by the user.

5. Which operating systems does Amazon EKS support?
Amazon Linux 2

Ubuntu

Windows Server (with Kubernetes 1.14 and later)

6. What is the Amazon EKS Connector?
The Amazon EKS Connector allows you to connect Kubernetes clusters running outside AWS to Amazon EKS, enabling centralized management and monitoring.

AWS CLI Example:
Register an external cluster with EKS.

aws eks register-cluster \
  --name ExternalCluster \
  --connector-config file://connector-config.json

7. Can I connect a cluster outside of an AWS Region?

Yes, using Amazon EKS Anywhere, you can connect and manage on-premises Kubernetes clusters.

8. How does Amazon EKS handle security?

Integration with IAM for authentication.

Network isolation using Amazon VPC.

Pod security policies for compliance.

Managed upgrades for control plane and worker nodes.

Terraform Block for Security Groups:

resource "aws_security_group" "eks_security_group" {
  name   = "eks-security-group"
  vpc_id = var.vpc_id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
9. What are the pricing models for Amazon EKS?

Control Plane: $0.10 per hour per cluster.

Worker Nodes: Charged based on EC2/Fargate usage.

EKS Anywhere: Separate licensing fees.

10. How do I get started with Amazon EKS using AWS CLI?

Create an EKS Cluster:

aws eks create-cluster \
  --name MyCluster \
  --role-arn <CLUSTER_ROLE_ARN> \
  --resources-vpc-config subnetIds=<SUBNET_IDS>,securityGroupIds=<SECURITY_GROUP_IDS>
  
Create Worker Nodes:

aws eks create-nodegroup \
  --cluster-name MyCluster \
  --nodegroup-name MyNodeGroup \
  --node-role <NODE_ROLE_ARN> \
  --subnets <SUBNET_IDS> \
  --scaling-config minSize=1,maxSize=3,desiredSize=2
Terraform Block for EKS Cluster:

resource "aws_eks_cluster" "example" {
  name     = "MyCluster"
  role_arn = aws_iam_role.eks_cluster_role.arn

  vpc_config {
    subnet_ids         = var.subnet_ids
    security_group_ids = [aws_security_group.eks_security_group.id]
  }
}
11. How do I upgrade my Kubernetes version in Amazon EKS?
Update the control plane:

aws eks update-cluster-version \
  --name MyCluster \
  --kubernetes-version 1.21
Update the worker nodes (manually or using managed node groups).

12. How does Amazon EKS handle high availability (HA)?

The EKS control plane is distributed across multiple AZs.

Worker nodes can be deployed in multiple AZs for redundancy.

13. How do I connect an on-premises Kubernetes cluster to Amazon EKS?

Use the EKS Connector to integrate on-premises Kubernetes clusters with Amazon EKS.

14. How does Amazon EKS integrate with other AWS services?

IAM: For secure authentication.

CloudWatch: For monitoring logs and metrics.

ALB/ELB: For traffic distribution.

15. What are best practices on EKS?

Use IAM Roles for Service Accounts (IRSA) for fine-grained access control.

Deploy worker nodes in multiple AZs.

Regularly update Kubernetes versions and worker nodes.

Use pod security policies and network policies.

16. What are Kubernetes health checks and why are they important?

Health checks monitor the state of your application and ensure only healthy pods receive traffic.

17. What are the different types of health checks in Kubernetes?

Liveness Probe: Checks if the application is running.

Readiness Probe: Checks if the application is ready to serve traffic.

18. How do you configure a liveness probe in a Kubernetes pod?
Example YAML:


livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5

19. What is the purpose of a readiness probe?

Readiness probes determine if a pod is ready to handle requests.


20. Can you explain the difference between liveness and readiness probes?

Liveness Probe: Checks the app's health; restarts the pod if it fails.

Readiness Probe: Ensures the pod is ready to receive traffic.

