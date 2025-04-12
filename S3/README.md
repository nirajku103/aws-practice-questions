# aws-practice-questions

Q1. What is S3?

Amazon Simple Storage Service (S3) is a highly scalable, durable, and secure object storage service that allows users to store and retrieve any amount of data at any time. It is ideal for use cases such as backup and restore, content distribution, data lakes, and big data analytics.

Real-Time Use Case: Hosting a static website on S3 to serve web pages, images, and other assets globally with high availability.

Q2. What are the various types of Buckets?

While all S3 buckets share the same basic functionality, they can be configured differently based on use cases:

Standard Bucket: Default configuration for general-purpose storage.

Versioned Bucket: Tracks changes to objects by maintaining older versions.

Logging Bucket: Stores access logs for another bucket.

Encrypted Bucket: Enforces server-side encryption (e.g., AES-256 or AWS KMS).

Replication Bucket: Used for Cross-Region Replication (CRR) or Same-Region Replication (SRR).

Q3. How does Amazon ensure High Availability (HA) for S3?

Amazon S3 achieves HA through:

Replication: Objects are replicated across multiple Availability Zones (AZs).

Durability: 99.999999999% (11 9’s) durability is maintained by storing multiple copies of data.

Failover Mechanisms: Automatic failover between AZs in case of hardware failures.

Real-Time Use Case: Storing business-critical documents and ensuring they are accessible during outages.

Q4. What is S3 Versioning and Explain Different States?

S3 Versioning is a feature that allows maintaining multiple versions of an object.

Unversioned: Default state where versioning is disabled.

Enabled: Maintains all versions of an object.

Suspended: Stops versioning but retains existing versions.

AWS CLI Command:

aws s3api put-bucket-versioning \
    --bucket my-bucket \
    --versioning-configuration Status=Enabled

Q5. What Does a Lifecycle Rule Consist Of?

A lifecycle rule defines actions to automatically transition or expire objects based on their age.

Transition Actions: Move objects to cheaper storage classes like Glacier.

Expiration Actions: Delete objects after a certain period.

Example Rule:

Transition objects to S3 Glacier after 30 days.

Permanently delete after 365 days.

Q6. How to Set an S3 Lifecycle Configuration Using AWS CLI?

Command:
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-bucket \
    --lifecycle-configuration file://lifecycle.json

{
    "Rules": [
        {
            "ID": "ArchiveAndExpire",
            "Status": "Enabled",
            "Filter": {},
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "GLACIER"
                }
            ],
            "Expiration": {
                "Days": 365
            }
        }
    ]
}

Q7. How Would You Prevent Versioning-Related Performance Degradation Issues?

Use lifecycle policies to archive or delete old versions.

Apply prefixes to optimize performance for buckets with a high number of objects.

Enable Intelligent-Tiering to optimize costs and performance.

Real-Time Use Case: Managing logs in a versioned bucket by automatically deleting old versions.

Q8. What Are the Different Storage Classes in S3?

S3 Standard: High-performance, low-latency.

S3 Intelligent-Tiering: Automatically moves objects between storage classes based on access patterns.

S3 Glacier: Long-term archival with slower retrieval.

S3 Glacier Deep Archive: Lowest-cost archival storage.

S3 One Zone-IA: Lower cost but stored in a single AZ.

Real-Time Use Case: Storing infrequently accessed backup files in Glacier to save costs.

Q9. How Does S3 Manage Costs, Meet Regulatory Requirements, and Reduce Latency?

Costs: Lifecycle policies, Intelligent-Tiering, and archival classes reduce storage costs.

Regulatory Requirements: Bucket policies and AWS Config ensure compliance with regulations like GDPR.

Latency: S3 Transfer Acceleration speeds up global uploads and downloads.

Q10. Why is Amazon S3 the Best Backend to Store Your State File?

Durability and Availability: Ensures state files are always accessible.

Versioning: Maintains history for rollbacks.

Encryption: Protects sensitive data.

Q11. Sample S3 Bucket Following Best Practices

Terraform Resource Block:

resource "aws_s3_bucket" "secure_bucket" {
  bucket = "my-secure-bucket"
  acl    = "private"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "aws:kms"
      }
    }
  }

  lifecycle_rule {
    id      = "archive_rule"
    enabled = true

    transition {
      days          = 30
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}

Q12. What Are the Key S3 Metrics Available in CloudWatch?

BucketSizeBytes: Total size of objects.

NumberOfObjects: Number of stored objects.

4xxErrorRate and 5xxErrorRate: HTTP error rates.

FirstByteLatency: Time for the first byte to be transferred.

Q13. How Does S3 Handle Access Logging?

Access logs record requests made to a bucket.

Logs are stored in another S3 bucket for analysis.

Enable Logging Using CLI:

aws s3api put-bucket-logging \
    --bucket source-bucket \
    --bucket-logging-status '{"LoggingEnabled":{"TargetBucket":"log-bucket","TargetPrefix":"log/"}}'

Q14. What Are Presigned URLs?

Presigned URLs allow users to grant temporary access to objects without exposing credentials.

AWS CLI Command:
aws s3 presign s3://my-bucket/my-object --expires-in 3600

Real-Time Use Case: Sharing a file temporarily with a client without making it public.

Q15. What Is S3 Select and How Is It Used?

S3 Select allows querying specific data from an object using SQL without downloading the entire object.

AWS CLI Command:
aws s3api select-object-content \
    --bucket my-bucket \
    --key my-file.csv \
    --expression "SELECT * FROM S3Object WHERE age > 30" \
    --expression-type SQL \
    --input-serialization file://input.json \
    --output-serialization file://output.json

Q16. What Is Cross-Region Replication (CRR) in S3?

CRR automatically replicates objects from one bucket to another in a different region.

resource "aws_s3_bucket_replication_configuration" "example" {
  bucket = aws_s3_bucket.source_bucket.id

  rule {
    id     = "ReplicationRule"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.destination_bucket.arn
      storage_class = "GLACIER"
    }
  }
}

Q17. Explain S3 Access Grants

Access grants include policies, ACLs, and IAM roles that define who can access a bucket or object and their permissions.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:user/example"},
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}