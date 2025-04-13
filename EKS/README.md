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

``aws eks register-cluster \
  --name ExternalCluster \
  --connector-config file://connector-config.json``

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

``aws eks create-cluster \
  --name MyCluster \
  --role-arn <CLUSTER_ROLE_ARN> \
  --resources-vpc-config subnetIds=<SUBNET_IDS>,securityGroupIds=<SECURITY_GROUP_IDS>``
  
Create Worker Nodes:

``aws eks create-nodegroup \
  --cluster-name MyCluster \
  --nodegroup-name MyNodeGroup \
  --node-role <NODE_ROLE_ARN> \
  --subnets <SUBNET_IDS> \
  --scaling-config minSize=1,maxSize=3,desiredSize=2``

Terraform Block for EKS Cluster:

resource "aws_eks_cluster" "eks-cluster" {
  name     = "demo-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn

  vpc_config {
    subnet_ids         = var.subnet_ids
    security_group_ids = [aws_security_group.eks_security_group.id]
  }
}
11. How do I upgrade my Kubernetes version in Amazon EKS?
Update the control plane:

``aws eks update-cluster-version \
  --name MyCluster \
  --kubernetes-version 1.21``
Update the worker nodes (manually or using managed node groups).

12. How does Amazon EKS handle high availability (HA)?

The EKS control plane is distributed across multiple AZs.

Worker nodes can be deployed in multiple AZs for redundancy.

13. How do I connect an on-premises Kubernetes cluster to Amazon EKS?

Use the EKS Connector to integrate on-premises Kubernetes clusters with Amazon EKS.

14. How does Amazon EKS integrate with other AWS services?

Amazon Elastic Kubernetes Service (EKS) integrates with a variety of other AWS services to provide enhanced functionality, security, monitoring, and scalability. Here's how EKS integrates with other key AWS services:

  1. Amazon EC2 (Elastic Compute Cloud)

EKS runs Kubernetes worker nodes (EC2 instances) within your Amazon Virtual Private Cloud (VPC). EC2 instances serve as the infrastructure on which Kubernetes pods are scheduled.

``aws autoscaling create-launch-configuration --launch-configuration-name my-launch-configuration \
  --image-id ami-xxxxxxxx \
  --instance-type t2.medium \
  --security-groups sg-xxxxxx \
  --key-name my-key-pair \
  --iam-instance-profile arn:aws:iam::1625347236:instance-profile/EKSWorkerNodeRole``


eg: EKS provisions EC2 instances for scaling worker nodes based on demand. It integrates with Auto Scaling Groups for automatic scaling of EC2 instances based on pod resource requirements.

  2. Amazon VPC (Virtual Private Cloud)

Integration: EKS clusters are deployed within a VPC to provide network isolation for your Kubernetes pods. The VPC integration enables Kubernetes pods to communicate securely within the same VPC or across VPCs.

``aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region us-west-2``
``aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.1.0/24 --availability-zone us-west-2a --region us-west-2``
``aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.2.0/24 --availability-zone us-west-2b --region us-west-2``


terraform block : 
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr_block
}

# Internet Gateway for Public Subnets
resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.public_subnets)
  vpc_id                  = aws_vpc.this.id
  cidr_block              = element(var.public_subnets, count.index)
  availability_zone       = element(var.public_subnet_azs, count.index)
  map_public_ip_on_launch = true
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.private_subnets)
  vpc_id           = aws_vpc.this.id
  cidr_block       = element(var.private_subnets, count.index)
  availability_zone = element(var.private_subnet_azs, count.index)
}

eg: VPC enables fine-grained control over networking, allowing you to configure network access control (e.g., security groups and network ACLs) to ensure secure pod communication.

  3. Amazon RDS (Relational Database Service)
Integration: EKS can be used to run applications that need access to databases hosted in Amazon RDS.

eg: Applications running on EKS clusters can interact with Amazon RDS databases (like MySQL, PostgreSQL, or Aurora) through private VPC connectivity, providing low-latency and secure access to relational databases.

  4. Amazon ECR (Elastic Container Registry)
Integration: EKS uses Amazon ECR for storing and managing Docker container images. EKS can pull container images from ECR repositories for deployment to the cluster.

eg: Developers push Docker images to ECR, and EKS pulls those images to run containers in the Kubernetes pods.

  5. AWS IAM (Identity and Access Management)
Integration: EKS integrates with IAM for role-based access control (RBAC). IAM roles are used to grant permissions to Kubernetes nodes, applications, and services for interacting with other AWS resources.

eg: EKS uses IAM for authenticating the Kubernetes control plane and managing access to AWS services. IAM roles are also used for service accounts, allowing Kubernetes workloads to interact with AWS services like S3, DynamoDB, and more.

  6. AWS CloudWatch
Integration: EKS integrates with Amazon CloudWatch for logging, monitoring, and alerting. Kubernetes control plane logs and container logs can be sent to CloudWatch for centralized monitoring.

eg: Use CloudWatch to monitor metrics like CPU usage, memory, and network traffic for both Kubernetes nodes and pods. CloudWatch logs can capture container logs, enabling you to track application logs and system-level logs for troubleshooting.

  7. AWS CloudTrail
Integration: EKS works with CloudTrail to log API calls and actions performed on your Kubernetes resources. CloudTrail captures management actions taken on the EKS control plane, as well as actions that interact with other AWS services.

eg: CloudTrail allows you to track who made changes to the EKS cluster and provides detailed logs for auditing and compliance purposes.

  8. Amazon Route 53
Integration: EKS integrates with Route 53 for DNS resolution, enabling services running within Kubernetes to be accessible via domain names.

eg: You can create DNS records for your services within EKS so that applications can be accessed via user-friendly URLs. Route 53 also enables load balancing and service discovery within the Kubernetes cluster.

  9. AWS Load Balancer (ALB, NLB)
Integration: EKS integrates with the AWS Application Load Balancer (ALB) and Network Load Balancer (NLB) for routing traffic to your Kubernetes services.

eg: EKS automatically provisions ALBs and NLBs for services exposed through Kubernetes Ingress or service type LoadBalancer. This ensures traffic is directed to the appropriate Kubernetes pods running on worker nodes.

  10. Amazon S3 (Simple Storage Service)
Integration: EKS applications can interact with Amazon S3 to store and retrieve objects. You can mount S3 buckets as volumes or use SDKs from within your application pods.

eg: Store application logs, backups, or static content in S3, and use Kubernetes workloads to manage this data.

  11. AWS Secrets Manager and AWS Systems Manager Parameter Store
Integration: EKS integrates with AWS Secrets Manager and Systems Manager Parameter Store to manage sensitive data, such as database credentials and API keys, securely.

eg: You can configure Kubernetes workloads to retrieve secrets (e.g., credentials) from Secrets Manager or Parameter Store at runtime, enabling secure management of sensitive information within your applications.

  12. AWS Lambda
Integration: EKS can trigger AWS Lambda functions through event-based mechanisms like Amazon CloudWatch Events or AWS Step Functions.

eg: You can use Lambda to trigger tasks or workflows based on events from EKS, such as changes to Kubernetes resources (e.g., when a pod fails, trigger a Lambda function for recovery).


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


``livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5``

19. What is the purpose of a readiness probe?

Readiness probes determine if a pod is ready to handle requests.


20. Can you explain the difference between liveness and readiness probes?

Liveness Probe: Checks the app's health; restarts the pod if it fails.

Readiness Probe: Ensures the pod is ready to receive traffic.

