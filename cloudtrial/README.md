1. What is AWS CloudTrail?

AWS CloudTrail records API calls made on your AWS account, allowing you to track user activity, resource changes, and security events.

2. What are the benefits of using AWS CloudTrail?

Auditing and monitoring API usage.

Security analysis and troubleshooting.

Compliance tracking.

3. How can I view my account activity with AWS CloudTrail?

You can view CloudTrail logs in the CloudTrail Console or use Amazon S3 to store and query logs.

4. What is a CloudTrail trail?

A CloudTrail trail is a configuration that enables CloudTrail to log API calls and deliver the logs to an S3 bucket.

5. Can I use AWS CloudTrail to monitor multiple AWS accounts?

Yes, CloudTrail supports multi-account logging by setting up trails for multiple accounts or using CloudTrail Organizations.

6. What is AWS CloudTrail Lake?

CloudTrail Lake enables advanced querying and analytics for CloudTrail logs across AWS accounts.

7. How long does AWS CloudTrail retain data?

CloudTrail retains logs for 90 days in CloudTrail Lake by default. You can configure S3 retention policies for longer storage.

8. What are CloudTrail Insights?

CloudTrail Insights helps identify unusual API activity and anomalies, such as sudden spikes in API calls.

9. How to set up a CloudTrail trail using AWS CLI?

aws cloudtrail create-trail \
    --name my-trail \
    --s3-bucket-name my-cloudtrail-bucket \
    --is-multi-region-trail

10. What are best practices for CloudTrail?

Enable multi-region trails.

Store logs in an S3 bucket with restricted access.

Enable CloudTrail Insights for anomaly detection.

11. What are CloudTrail log file formats?

CloudTrail logs are stored in JSON format, which includes metadata such as the event name, timestamp, user, and resource details.