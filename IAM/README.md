Q1. What is AWS Identity and Access Management (IAM)?

AWS IAM is a service that enables secure control of access to AWS resources. It allows you to manage users, groups, and permissions, ensuring that resources are only accessed by authorized entities.

Real-Time Use Case: Granting developers read-only access to specific S3 buckets while providing administrators full access.

Q2. How Does IAM Work and What Can I Do With It?

IAM works by managing users, roles, and policies to control who can access AWS resources and what actions they can perform.

Capabilities:

Create and manage users and groups.

Assign permissions using policies.

Enable temporary access with roles.

eg: Using IAM roles to grant Lambda functions permission to access an S3 bucket.

Q3. What Are Least-Privilege Permissions?

The principle of least privilege means granting only the permissions necessary to perform a specific task.

``aws iam attach-role-policy \
    --role-name ExampleRole \
    --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccessToS3``

Least-Pivilage basically means kind of read-only-access

Q4. What Are IAM Policies?

IAM policies define permissions to AWS resources. They can be attached to users, groups, or roles.

Example Policy:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::niraj-bucket"
    }
  ]
}

Q5. Why Should I Use IAM Roles?

IAM roles provide temporary credentials, improving security by avoiding hard-coded credentials.

eg: Assigning an IAM role to an EC2 instance to allow it to upload logs to an S3 bucket without using access keys.

Q6. How to Create Policies Using AWS CLI?

Command:
``aws iam create-policy \
    --policy-name ExamplePolicy \
    --policy-document file://policy.json``

sample-policy.json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}

Q7. What Types of Policies?

Managed Policies: Predefined policies by AWS or custom policies created by users. bascially read only policies

``aws iam attach-user-policy \
    --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess \
    --user-name niraj-react``


Inline Policies: Policies embedded directly into an IAM entity. eg : Allow a specific IAM role to manage a specific S3 bucket.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}

Attaching this policy to the role

aws iam put-role-policy \
    --role-name BucketAccessRole \
    --policy-name ManageSpecificS3Bucket \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::niraj-bucket/*"
            }
        ]
    }'


Service Control Policies (SCPs): Govern permissions for AWS Organizations.

``aws organizations create-policy \
    --content '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Deny",
                "Action": "s3:CreateBucket",
                "Resource": "*"
            }
        ]
    }' \
    --name "DenyS3BucketCreation" \
    --type "SERVICE_CONTROL_POLICY"``

  ``aws organizations attach-policy \
    --policy-id <SCP_ID> \
    --target-id <OU_ID>``


eg : Attaching SCPs to an OU to prevent the creation of S3 buckets in specific accounts.

Q8. What Are the Best Practices?

Enable MFA for privileged users.

Grant least privilege.

Rotate credentials regularly.

Use roles instead of access keys.

Audit and monitor IAM activity.

Q9. Which Policy Attached to Role to Perform ECS Task?

To perform ECS tasks (e.g., running and stopping tasks), you need to attach an IAM policy with the necessary permissions to the IAM role associated with ECS Task Execution Role.

Required Policy:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:RunTask",
        "ecs:StopTask"
      ],
      "Resource": "*"
    }
  ]
}

Terraform block :

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

Attach the ecs tasks policy to the task execution role

resource "aws_iam_role_policy_attachment" "custom_ecs_task_policy_attachment" {
  role       = aws_iam_role.ecs_task_execution_role.name
  policy_arn = aws_iam_policy.ecs_task_policy.arn
}


Q10. How to Assign User/Admin IAM Policy for ECS Management?

Command:

``aws iam attach-user-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess \
    --user-name AdminUser``

Q11. If You're Creating and Managing AWS EKS Service, Which IAM Policies Are Required?

AmazonEKSClusterPolicy

AmazonEKSWorkerNodePolicy

AmazonEC2ContainerRegistryReadOnly

AmazonEKSFargatePodExecutionRolePolicy (if using compute as fargate)

AmazonEKSServicePolicy

AmazonEKSVPCResourceController


Terraform Block:

resource "aws_iam_role_policy_attachment" "eks_policy" {
  role       = aws_iam_role.eks_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:CreateCluster",
        "eks:DescribeCluster",
        "eks:DeleteCluster",
        "eks:UpdateClusterVersion",
        "eks:TagResource"
      ],
      "Resource": "*"
    }
  ]
}


Q12. What Is the Use of iam:PassRole Permission?

The iam:PassRole permission allows passing an IAM role to an AWS service.
It is essential for actions where an IAM role needs to be assumed by an AWS service to perform specific tasks on behalf of the user

you need to allow an EC2 instance to assume a specific role. For example, when launching an EC2 instance that needs to assume a role to access resources like S3 or DynamoDB, the user launching the instance must have the iam:PassRole permission to pass the IAM role to the EC2 instance

resource "aws_instance" "ec2_instance" {
  ami           = "ami-0c55b159cbfafe1f0" # Example AMI ID
  instance_type = "t2.micro"
  iam_instance_profile = aws_iam_role.ec2_s3_role.name
}


Q13. What Is a Trust Policy?

A trust policy specifies which entities can assume a role.

Example:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

Q14. Can You Restrict IAM User Access by IP Address?

Yes, using a condition in the policy.

Example Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": "192.0.2.0/24"
        }
      }
    }
  ]
}
Q15. How to Audit IAM Changes?

Enable CloudTrail: Tracks all IAM API activities.

Command:
``aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser``

Q16. How Would You Restrict an IAM Role to Access S3 Only If MFA Is Enabled and the Request Comes from a Specific VPC?

We can use an IAM policy with conditions based on both MFA authentication and VPC conditions

Policy:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::example-bucket/*",
      "Condition": {
        "Bool": { "aws:MultiFactorAuthPresent": "true" },
        "StringEquals": { "aws:SourceVpc": "vpc-1a2b3c4d" }
      }
    }
  ]
}

Q17. What IAM Roles Are Typically Used for Each AWS Service?

S3: Role with S3 read/write permissions.

ECS: Role with ecs:RunTask and ecs:StopTask permissions.

EKS: Role with eks:DescribeCluster and eks:CreateNodegroup permissions.

Lambda: Role with permissions for triggering services.

ALB: Role for ALB ingress controller with elasticloadbalancing:* permissions.

EC2: Role with access to required resources like S3 or DynamoDB.

Example Terraform Block for Lambda Role:
resource "aws_iam_role" "lambda_role" {
  name = "lambda-role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
}

resource "aws_iam_policy_attachment" "lambda_policy" {
  name       = "lambda-policy"
  roles      = [aws_iam_role.lambda_role.name]
  policy_arn = "arn:aws:iam::aws:policy/AWSLambdaExecute"
}