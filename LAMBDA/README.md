1. What is AWS Lambda?

AWS Lambda is a serverless compute service that allows you to run code without provisioning or managing servers. You simply upload your code, and Lambda handles the rest, such as scaling and managing the compute infrastructure.

2. What events can trigger an AWS Lambda function?

AWS Lambda can be triggered by events from various AWS services, such as:

S3: Object creation

DynamoDB: Stream updates

API Gateway: HTTP requests

SNS: Notifications

SQS: Message queues

CloudWatch Events: Scheduled triggers

Custom Applications: Using SDKs or HTTP APIs

Example AWS CLI Command to Add S3 Trigger:

``aws s3api put-bucket-notification-configuration \
  --bucket my-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [
      {
        "LambdaFunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:my-function",
        "Events": ["s3:ObjectCreated:*"]
      }
    ]
  }'``

Terraform Resource Block:

resource "aws_lambda_function" "s3_lambda" {
  function_name = "s3-event-processor"
  runtime       = "python3.8"
  role          = aws_iam_role.lambda_exec.arn
  handler       = "handler.lambda_handler"
  source_code_hash = filebase64sha256("lambda.zip")
  filename      = "lambda.zip"
}

resource "aws_s3_bucket_notification" "bucket_notification" {
  bucket = aws_s3_bucket.bucket.id

  lambda_function {
    lambda_function_arn = aws_lambda_function.s3_lambda.arn
    events              = ["s3:ObjectCreated:*"]
  }
}

3. How does AWS Lambda handle scaling?

Lambda automatically scales horizontally by creating new instances of the function to handle incoming requests. Each invocation is handled in its own execution environment.

4. Can Lambda functions run on a timed schedule?

Yes, AWS Lambda can run on a timed schedule using Amazon EventBridge (formerly CloudWatch Events).

AWS CLI Example:

``aws events put-rule \
  --schedule-expression "rate(5 minutes)" \
  --name MyScheduledRule``

``aws lambda add-permission \
  --function-name my-function \
  --statement-id MyScheduledEvent \
  --action "lambda:InvokeFunction" \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-west-2:123456789012:rule/MyScheduledRule``

``aws events put-targets \
  --rule MyScheduledRule \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-west-2:123456789012:function:my-function"``

Terraform Resource Block:

resource "aws_cloudwatch_event_rule" "schedule" {
  name                = "my-scheduled-event"
  schedule_expression = "rate(5 minutes)"
}

resource "aws_cloudwatch_event_target" "lambda_target" {
  rule      = aws_cloudwatch_event_rule.schedule.name
  target_id = "lambda"
  arn       = aws_lambda_function.my_lambda.arn
}

resource "aws_lambda_permission" "allow_eventbridge" {
  statement_id  = "AllowEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.my_lambda.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.schedule.arn
}

5. What is the maximum execution duration for a Lambda function?

The maximum execution duration for a Lambda function is 15 minutes (900 seconds).

6. How can a Lambda function retain state between invocations?

AWS Lambda is stateless by design. To retain state:

Use external services like DynamoDB, RDS, or S3.

Use AWS Lambda layers or environment variables for caching purposes.

7. What are some common use cases for AWS Lambda?

Real-time file processing (e.g., image or video processing).

Data transformation pipelines.

Backend services for mobile and web apps.

Serverless APIs.

Scheduled tasks.

8. What are the best practices for AWS Lambda?

Keep functions small and focused.

Use environment variables for configuration.

Minimize package size to reduce cold start time.

Leverage provisioned concurrency to mitigate cold starts.

Optimize memory and timeout settings.

Use proper IAM roles and least privilege.

9. How do you create an AWS Lambda function using the AWS CLI?

AWS CLI Example:

``aws lambda create-function \
  --function-name my-function \
  --runtime nodejs14.x \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip``

10. How can you update the code of an existing Lambda function using the AWS CLI?
AWS CLI Example:

``aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip``

11. How do you add permissions to an AWS Lambda function to allow it to be triggered by an S3 event?

AWS CLI Example:

``aws lambda add-permission \
  --function-name my-function \
  --statement-id s3invoke \
  --action "lambda:InvokeFunction" \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-bucket``

12. How can you monitor AWS Lambda function performance, what important metrics to capture?
Use Amazon CloudWatch to monitor metrics like:

Invocations

Errors

Duration

Throttles

Concurrency

Terraform Resource Block:

resource "aws_cloudwatch_metric_alarm" "lambda_error_alarm" {
  alarm_name          = "lambda-error-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = 60
  statistic           = "Sum"
  threshold           = 1
  dimensions = {
    FunctionName = aws_lambda_function.my_lambda.function_name
  }
}

13. How can you use AWS Lambda to process data from an Amazon S3 bucket?
Create a trigger for object-created events.

Process the data in the Lambda handler.

AWS CLI Example:

``aws s3api put-bucket-notification-configuration \
  --bucket my-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [
      {
        "LambdaFunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:my-function",
        "Events": ["s3:ObjectCreated:*"]
      }
    ]
  }'``

14. How can you use AWS Lambda to respond to HTTP requests?

Use API Gateway to trigger Lambda functions for HTTP requests.

Terraform Resource Block:

resource "aws_apigatewayv2_api" "http_api" {
  name          = "my-http-api"
  protocol_type = "HTTP"
}

resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.my_lambda.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.http_api.execution_arn}/*"
}

15. How can you use AWS Lambda to interact with Amazon DynamoDB?
Attach a DynamoDB trigger to the Lambda function to process stream records.

Terraform Resource Block:

resource "aws_dynamodb_table" "my_table" {
  name           = "my-table"
  hash_key       = "id"
  stream_enabled = true
  stream_view_type = "NEW_AND_OLD_IMAGES"

  attribute {
    name = "id"
    type = "S"
  }
}

resource "aws_lambda_event_source_mapping" "dynamodb_trigger" {
  event_source_arn = aws_dynamodb_table.my_table.stream_arn
  function_name    = aws_lambda_function.my_lambda.function_name
}

16. What is a cold start in AWS Lambda?

A cold start happens when AWS initializes a new execution environment for a Lambda function due to lack of warm containers or low provisioned concurrency.

17. Why do cold starts happen in AWS Lambda?

Cold starts occur when:

A function is invoked after being idle.

Scaling occurs due to new requests.

18. How can I reduce the impact of cold starts in AWS Lambda?

Use provisioned concurrency.

Optimize dependencies to reduce initialization time.

Use lighter runtimes like Node.js or Python.

Terraform Resource Block for Provisioned Concurrency:

resource "aws_lambda_provisioned_concurrency_config" "my_function_concurrency" {
  function_name          = aws_lambda_function.my_lambda.function_name
  provisioned_concurrent_executions = 5
}