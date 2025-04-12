1. What is Amazon CloudWatch?

Amazon CloudWatch is a monitoring and observability service that provides metrics, logs, and alarms for AWS resources and applications.

2. What can I use to access CloudWatch?

You can access CloudWatch through the AWS Management Console, AWS CLI, CloudWatch SDKs, or CloudWatch APIs.

3. What is Amazon CloudWatch Logs?

CloudWatch Logs allows you to monitor, store, and access log files from AWS resources (like EC2 instances, Lambda functions, and VPC Flow Logs).

4. What access management policies can I implement for CloudWatch?

You can control access to CloudWatch resources using IAM policies. For example, you can allow users to view metrics, set up alarms, or manage CloudWatch Logs.

IAM Policy Example:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:DescribeAlarms",
                "cloudwatch:GetDashboard",
                "cloudwatch:ListMetrics"
            ],
            "Resource": "*"
        }
    ]
}

5. How does Amazon CloudWatch collect data?

CloudWatch collects data through:

CloudWatch Metrics from AWS resources and custom metrics.

CloudWatch Logs from applications and AWS services.

CloudWatch Events to capture events and trigger actions.

6. What are CloudWatch Alarms?

CloudWatch Alarms allow you to monitor metrics and trigger actions (such as notifications) when thresholds are breached.

7. Can I use CloudWatch to monitor my on-premises resources?

Yes, you can use CloudWatch Agent to send logs and metrics from on-premises servers to CloudWatch.

8. What is CloudWatch Events?

CloudWatch Events captures changes to your AWS resources and triggers automated actions based on event patterns.

9. How long does CloudWatch retain data?

CloudWatch retains metrics data for 15 months. Logs can be retained based on retention settings you configure.

10. What are best practices for using AWS CloudWatch effectively?

Use CloudWatch Alarms to set thresholds and trigger automated actions.

Enable CloudWatch Logs to capture detailed data from AWS resources.

Set up custom metrics to monitor application-specific performance.

11. What are key metrics and make list for monitoring ECS, EKS, Lambda, and ALB in AWS CloudWatch like container metrics etc.

ECS Metrics:

CPUUtilization

MemoryUtilization

NetworkBytes

EKS Metrics:

NodeCPUUtilization

NodeMemoryUtilization

Lambda Metrics:

Invocations

Errors

Duration

ALB Metrics:

RequestCount

TargetResponseTime

HTTPCode_ELB_5XX

