1. What is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that enables you to run and scale containerized applications on AWS. It supports Docker containers and allows developers to deploy applications without managing the underlying infrastructure.

2. What are the different launch types in Amazon ECS?

Fargate Launch Type: Allows running containers without managing the underlying EC2 instances.

EC2 Launch Type: Uses ECS-optimized Amazon EC2 instances for running containers.

External Launch Type: Allows running tasks on your on-premises infrastructure.

AWS CLI Example:
Create a cluster with a specified launch type.

aws ecs create-cluster --cluster-name MyCluster --capacity-providers FARGATE
Terraform Block:

resource "aws_ecs_cluster" "example" {
  name = "MyCluster"
}
3. How does Amazon ECS ensure high availability (HA)?
Amazon ECS ensures high availability by distributing tasks across multiple Availability Zones (AZs) and leveraging the AWS Service Scheduler to replace unhealthy tasks or instances.

Key Features:

Multi-AZ deployment.

Integration with Elastic Load Balancing (ELB) for traffic distribution.

AWS CLI Example:
Configure a service with high availability using an ALB.

aws ecs create-service \
  --cluster MyCluster \
  --service-name MyService \
  --task-definition MyTaskDefinition \
  --load-balancers targetGroupArn=<TARGET_GROUP_ARN>,containerName=<CONTAINER_NAME>,containerPort=<CONTAINER_PORT> \
  --desired-count 2
  
4. What is a Task Definition in Amazon ECS?
A Task Definition is a blueprint for your application. It defines which containers to use, resources like CPU and memory, networking mode, and environment variables.

AWS CLI Example:
Register a Task Definition.

aws ecs register-task-definition \
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
  ]'
Terraform Block:

resource "aws_ecs_task_definition" "example" {
  family                   = "MyTaskFamily"
  container_definitions    = <<DEFINITION
[
  {
    "name": "web",
    "image": "nginx",
    "cpu": 256,
    "memory": 512,
    "essential": true
  }
]
DEFINITION
  network_mode             = "bridge"
}
5. What is a Service in Amazon ECS?
A Service manages long-running tasks on your cluster and can integrate with load balancers for high availability.

AWS CLI Example:
Create a Service.

aws ecs create-service \
  --cluster MyCluster \
  --service-name MyService \
  --task-definition MyTaskDefinition \
  --desired-count 3
6. How can I update a running service in Amazon ECS?
AWS CLI Example:
Update a Service.

aws ecs update-service \
  --cluster MyCluster \
  --service MyService \
  --task-definition MyTaskDefinition \
  --desired-count 4
7. What are the key metrics available in CloudWatch for Amazon ECS?
CPU and memory utilization.

Task and service counts.

Network I/O metrics.

AWS CLI Example:
Get ECS metrics using CloudWatch.

aws cloudwatch get-metric-data \
  --metric-data-queries '[{"Id":"cpuUsage","MetricStat":{"Metric":{"Namespace":"AWS/ECS","MetricName":"CPUUtilization"},"Period":60,"Stat":"Average"}}]' \
  --start-time 2025-04-10T00:00:00Z \
  --end-time 2025-04-11T00:00:00Z
8. How does Amazon ECS handle logging?
ECS integrates with AWS CloudWatch Logs to capture container logs.

AWS CLI Example:
Enable logging in a Task Definition.

bash
Copy
Edit
aws ecs register-task-definition \
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
  ]'

9. What are IAM roles for tasks in Amazon ECS?
IAM roles for tasks allow containers to access AWS services securely.

AWS CLI Example:
Attach an IAM role to a Task Definition.

aws ecs register-task-definition \
  --family MyTaskFamily \
  --task-role-arn arn:aws:iam::123456789012:role/MyTaskRole \
  --container-definitions '[
    {
      "name": "web",
      "image": "nginx"
    }
  ]'
Terraform Block:

resource "aws_iam_role" "task_execution_role" {
  name = "MyTaskRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action    = "sts:AssumeRole",
        Effect    = "Allow",
        Principal = { Service = "ecs-tasks.amazonaws.com" }
      }
    ]
  })
}
10. What is ECS Service Auto Scaling?
It automatically adjusts the number of tasks in response to load changes.

AWS CLI Example:
Set up Auto Scaling.

aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/MyCluster/MyService \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 10

aws application-autoscaling put-scaling-policy \
  --policy-name MyScalePolicy \
  --service-namespace ecs \
  --resource-id service/MyCluster/MyService \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://policy.json
11. What is the difference between Amazon ECS and Amazon EKS?
Feature	ECS	EKS
Management	Fully managed by AWS	Kubernetes-managed
Complexity	Easier to use	More complex
Container Support	Docker	Kubernetes
12. How does Amazon ECS integrate with AWS Fargate?
Fargate runs containers without managing servers.

AWS CLI Example:
Run a task with Fargate.

aws ecs run-task \
  --cluster MyCluster \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345],assignPublicIp=ENABLED}" \
  --task-definition MyTaskDefinition
13. What are ECS Clusters?
A cluster is a logical grouping of tasks or services.

AWS CLI Example:

aws ecs create-cluster --cluster-name MyCluster
Terraform Block:

hcl
Copy
Edit
resource "aws_ecs_cluster" "example" {
  name = "MyCluster"
}


