1. What is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that enables you to run and scale containerized applications on AWS. It supports Docker containers and allows developers to deploy applications without managing the underlying infrastructure.

2. What are the different launch types in Amazon ECS?

Fargate Launch Type: Allows running containers without managing the underlying EC2 instances.

EC2 Launch Type: Uses ECS-optimized Amazon EC2 instances for running containers.

External Launch Type: Allows running tasks on your on-premises infrastructure.

AWS CLI Example:

Create a cluster with a specified launch type.

``aws ecs create-cluster --cluster-name MyCluster --capacity-providers FARGATE``
Terraform Block:

resource "aws_ecs_cluster" "this" {
  name = var.cluster_name
}

3. How does Amazon ECS ensure high availability (HA)?

Amazon ECS ensures high availability by distributing tasks across multiple Availability Zones (AZs) and leveraging the AWS Service Scheduler to replace unhealthy tasks or instances.

Key Features:

Multi-AZ deployment.

Integration with Elastic Load Balancing (ELB) for traffic distribution.

AWS CLI Example:

Configure a service with high availability using an ALB.

``aws ecs create-service \
  --cluster MyCluster \
  --service-name MyService \
  --task-definition MyTaskDefinition \
  --load-balancers targetGroupArn=<TARGET_GROUP_ARN>,containerName=<CONTAINER_NAME>,containerPort=<CONTAINER_PORT> \
  --desired-count 2``
  
4. What is a Task Definition in Amazon ECS?

A Task Definition is a blueprint for your application. It defines which containers to use, resources like CPU and memory, networking mode, and environment variables.

AWS CLI Example:
Register a Task Definition.

``aws ecs register-task-definition \
  --family MyTaskFamily \
  --network-mode bridge \
  --container-definitions '[
    {
      "name": "web",
      "image": "nginx",
      "cpu": 256,
      "memory": 512,
      "essential": true
    }
  ]'``

Terraform Block:

resource "aws_ecs_task_definition" "service1" {
  family                   = "${var.cluster_name}-service1"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
  execution_role_arn       = aws_iam_role.ecs_task_execution_role.arn
  task_role_arn            = aws_iam_role.ecs_task_role.arn

  container_definitions = jsonencode([
    {
      name      = "service1-container"
      image     = var.service1_image
      cpu       = 256
      memory    = 512
      essential = true
      portMappings = [{
        containerPort = 3001
        hostPort      = 3001
      }]
    }
  ])
}

5. What is a Service in Amazon ECS?
A Service manages long-running tasks on your cluster and can integrate with load balancers for high availability.

AWS CLI Example:
Create a Service.

``aws ecs create-service \
  --cluster MyCluster \
  --service-name MyService \
  --task-definition MyTaskDefinition \
  --desired-count 3``

6. How can I update a running service in Amazon ECS?
AWS CLI Example:
Update a Service.

``aws ecs update-service \
  --cluster MyCluster \
  --service MyService \
  --task-definition MyTaskDefinition \
  --desired-count 4``

7. What are the key metrics available in CloudWatch for Amazon ECS?
CPU and memory utilization.

Task and service counts.

Network I/O metrics.

AWS CLI Example:
Get ECS metrics using CloudWatch.

``aws cloudwatch get-metric-data \
  --metric-data-queries '[{"Id":"cpuUsage","MetricStat":{"Metric":{"Namespace":"AWS/ECS","MetricName":"CPUUtilization"},"Period":60,"Stat":"Average"}}]' \
  --start-time 2025-04-10T00:00:00Z \
  --end-time 2025-04-11T00:00:00Z``

8. How does Amazon ECS handle logging?

ECS integrates with AWS CloudWatch Logs to capture container logs.

AWS CLI Example:
Enable logging in a Task Definition.

``aws ecs register-task-definition \
  --family MyTaskFamily \
  --container-definitions '[
    {
      "name": "web",
      "image": "nginx",
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/MyTaskFamily",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "web"
        }
      }
    }
  ]'`` 

9. What are IAM roles for tasks in Amazon ECS?

IAM roles for tasks allow containers to access AWS services securely.

ECS Tasks role

resource "aws_iam_role" "ecs_task_role" {
  name = "${var.cluster_name}-ecs-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}


ECS Task Ecexution ROle

resource "aws_iam_role" "ecs_task_execution_role" {
  name = "${var.cluster_name}-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}


AWS CLI Example:
Attach an IAM role to a Task Definition.

``aws ecs register-task-definition \
  --family MyTaskFamily \
  --task-role-arn arn:aws:iam::123456789012:role/MyTaskRole \
  --container-definitions '[
    {
      "name": "web",
      "image": "nginx"
    }
  ]'``

10. What is ECS Service Auto Scaling?

It automatically adjusts the number of tasks in response to load changes.

AWS CLI Example:
Set up Auto Scaling.

``aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/MyCluster/MyService \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 10``

``aws application-autoscaling put-scaling-policy \
  --policy-name MyScalePolicy \
  --service-namespace ecs \
  --resource-id service/MyCluster/MyService \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://policy.json``

11. What is the difference between Amazon ECS and Amazon EKS?

ECS :

AWS-managed container orchestration service, No control plane to manage, fully managed by AWS

ECS manages clusters internally.

FOr worked noeds they have EC2 instances (or AWS Fargate for serverless).
Deep AWS integration with VPCs.
For monitoring we have cloud watch
DNS routing using load balancers AWS Application/Network Load Balancers.
No extra charge for ECS control plane; pay for underlying resources (EC2, Fargate).

EKS:

Fully managed Kubernetes control plane, Kubernetes API for cluster management.
EC2 instances (or AWS Fargate for serverless).
Integrates with AWS VPC CNI plugin.
For monitoring tools like promrtheus grafana can be implemented
Routing using Kubernetes Ingress with AWS ALB/ELB.
Flat fee for control plane + pay for underlying resources.


12. How does Amazon ECS integrate with AWS Fargate?

configure the provider and region in Terraform.

provider "aws" {
  region = "us-west-2"
}

create the ECS cluster where the containers will be orchestrated. This acts as the foundation for running and managing tasks and services.

resource "aws_ecs_cluster" "ecs_cluster" {
  name = "ecs-fargate-cluster"
}

To allow ECS to interact with AWS services (e.g., pulling container images, writing logs to CloudWatch), a task execution role needs to be created. This includes defining a role and attaching the required permissions

resource "aws_iam_role" "ecs_task_execution_role" {
  name = "ecs_task_execution_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

Attach the policies permissions to the role

resource "aws_iam_role_policy_attachment" "ecs_task_execution" {
  role       = aws_iam_role.ecs_task_execution_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

Create an ECS Task Definition

resource "aws_ecs_task_definition" "fargate_task" {
  family                   = "fargate-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"  # 0.25 vCPU
  memory                   = "512"  # 512 MB
  execution_role_arn       = aws_iam_role.ecs_task_execution_role.arn

  container_definitions = jsonencode([{
    name      = "fargate-container"
    image     = "nginx:latest"
    essential = true
    portMappings = [{
      containerPort = 80
      hostPort      = 80
    }]
  }])
}

create vpc with public and private subnet

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


create sg's

resource "aws_security_group" "ecs_sg" {
  vpc_id = aws_vpc.vpc.id

  ingress {
    from_port   = 80
    to_port     = 80
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

create ecs service to deploy

resource "aws_ecs_service" "service1" {
  name            = "${var.cluster_name}-service1"
  cluster         = aws_ecs_cluster.this.id
  task_definition = aws_ecs_task_definition.service1.arn
  desired_count   = var.service1_desired_count
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = var.subnets
    security_groups = [aws_security_group.ecs_service.id]
    assign_public_ip = true
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.service1.arn
    container_name   = "service1-container"
    container_port   = 3001
  }
}


Fargate runs containers without managing servers.

AWS CLI Example:
Run a task with Fargate.

``aws ecs run-task \
  --cluster MyCluster \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345],assignPublicIp=ENABLED}" \
  --task-definition MyTaskDefinition``



13. What are ECS Clusters?

A cluster is a logical grouping of tasks or services.

AWS CLI Example:

``aws ecs create-cluster --cluster-name NirajCluster``

Terraform Block:

resource "aws_ecs_cluster" "example" {
  name = "NirajCluster"
}


