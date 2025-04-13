1. What is an Application Load Balancer (ALB)?

An Application Load Balancer (ALB) is a service by AWS that automatically distributes incoming traffic across multiple targets (e.g., EC2 instances, containers, IP addresses) within one or more Availability Zones. It operates at the application layer (Layer 7) and supports advanced routing capabilities, including host-based and path-based routing.

2. How does ALB handle SSL termination?

SSL termination occurs when the ALB decrypts HTTPS traffic, reducing the workload on backend targets.

Steps for SSL Termination:

Create an SSL certificate (e.g., from AWS Certificate Manager).

resource "aws_acm_certificate" "ssl_cert" {
  domain_name       = "example.com"
  validation_method = "DNS"

  tags = {
    Environment = "production"
  }
}

Create the http listenetr with port 443

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.example.arn
  port              = 443
  protocol          = "HTTPS"

  ssl_policy = "ELBSecurityPolicy-2016-08"
  certificate_arn = aws_acm_certificate.ssl_cert.arn

  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.example.arn
  }
}


Configure an target group on the ALB.

resource "aws_lb_target_group" "example" {
  name     = "example-target-group"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.example.id
}

add the backends target to forward port

resource "aws_lb_target_group_attachment" "example" {
  target_group_arn = aws_lb_target_group.example.arn
  target_id        = aws_instance.backend.id
  port             = 80
}

AWS CLI Example:

``aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=<CERTIFICATE_ARN> \
  --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>``

3. How does ALB perform health checks?

ALB performs health checks by periodically sending requests to a specific endpoint on the registered targets to ensure their availability.

Default Parameters: Protocol, Path, Interval, Timeout, and Unhealthy/Healthy thresholds.

AWS CLI Example:

aws elbv2 modify-target-group \
  --target-group-arn <TARGET_GROUP_ARN> \
  --health-check-protocol HTTP \
  --health-check-path "/health" \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 3 \
  --unhealthy-threshold-count 3

Terraform Resource Block:

resource "aws_lb_target_group" "target_group" {
  name        = "my-target-group"
  protocol    = "HTTP"
  port        = 80
  vpc_id      = var.vpc_id

  health_check {
    path                = "/health"
    protocol            = "HTTP"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 3
    unhealthy_threshold = 3
  }
}

4. What is path-based routing in ALB?

Path-based routing allows you to direct requests to different target groups based on the URL path (e.g., /api or /static).

AWS CLI Example:

aws elbv2 create-rule \
  --listener-arn <LISTENER_ARN> \
  --priority 10 \
  --conditions '[{"Field":"path-pattern","PathPatternConfig":{"Values":["/api/*"]}}]' \
  --actions '[{"Type":"forward","TargetGroupArn":"<TARGET_GROUP_ARN>"}]'

Terraform Resource Block:

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = "80"
  protocol          = "HTTP"
  default_action {
    type = "fixed-response"
    fixed_response {
      content_type = "text/plain"
      message_body = "404: Not Found"
      status_code  = "404"
    }
  }
}

resource "aws_lb_listener_rule" "service1" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.service1.arn
  }

  condition {
    path_pattern {
      values = ["/health"]
    }
  }
}


5. How can I secure traffic between my ALB and my targets?

Use HTTPS for both ALB listeners and target group communication.
Enable TLS encryption at the target level.
Configure SSL/TLS for Backend Targets
Update the ALB Target Group Protocol to HTTPS

Update the Target Group to Use HTTPS

``aws elbv2 modify-target-group --target-group-arn <TARGET_GROUP_ARN> --protocol HTTPS --port 443``

Add an HTTPS Health Check

``aws elbv2 modify-target-group --target-group-arn <TARGET_GROUP_ARN> --health-check-protocol HTTPS --health-check-path /health``

Register Targets

``aws elbv2 register-targets --target-group-arn <TARGET_GROUP_ARN> --targets Id=<INSTANCE_ID>,Port=443``


6. How do I configure logging for my ALB?

Logging can be enabled by storing access logs in an S3 bucket.

AWS CLI Example:

``aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn <ALB_ARN> \
  --attributes Key=access_logs.s3.enabled,Value=true \
               Key=access_logs.s3.bucket,Value=<BUCKET_NAME> \
               Key=access_logs.s3.prefix,Value=<PREFIX>``

Terraform Resource Block:

resource "aws_lb" "alb" {
  name               = "my-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = var.subnet_ids

  enable_deletion_protection = true

  access_logs {
    bucket = aws_s3_bucket.alb_logs.bucket
    prefix = "alb"
    enabled = true
  }
}

7. Can I configure multiple listeners on an ALB?

Yes, ALBs support multiple listeners (e.g., HTTP and HTTPS).

AWS CLI Example:

aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>

# http listener

resource "aws_lb_listener" "http_listener" {
  load_balancer_arn = aws_lb.my_alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.http_target_group.arn
  }
}

# https listener

resource "aws_lb_listener" "https_listener" {
  load_balancer_arn = aws_lb.my_alb.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.my_certificate.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.https_target_group.arn
  }
}


8. How can I monitor my Application Load Balancer?

You can monitor your ALB using CloudWatch metrics, access logs, and health check statuses.

Key Metrics:

RequestCount

TargetResponseTime

HealthyHostCount

UnhealthyHostCount

9. How to enable monitoring for your ALB using the AWS Management Console and AWS CLI?

AWS CLI:
Enable CloudWatch metrics.

aws elbv2 describe-load-balancer-attributes \
  --load-balancer-arn <ALB_ARN>
Terraform Resource Block:

resource "aws_cloudwatch_metric_alarm" "high_request_count" {
  alarm_name          = "HighRequestCount"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "RequestCount"
  namespace           = "AWS/ApplicationELB"
  period              = 60
  statistic           = "Sum"
  threshold           = 1000
  dimensions = {
    LoadBalancer = aws_lb.alb.arn_suffix
  }
}
10. What are the best practices for ALB?

Enable HTTPS: Use HTTPS for secure communication.

Access Logs: Enable access logging for auditing.

Health Checks: Configure appropriate health check endpoints.

Security Groups: Restrict inbound traffic to ALB.

Path-Based Routing: Optimize routing using path- or host-based rules.

Autoscaling: Integrate with Target Group Auto Scaling.

