# Object Storage

**Object Storage** provides scalable, highly durable, S3-compatible cloud storage for unstructured data. Whether you are storing images and videos for a web application, holding database backup archives, or processing data pipelines, Object Storage gives you virtually unlimited storage capacity accessible from anywhere over HTTPS.

Because Superphenix Object Storage fully implements the standard **AWS S3 API**, you can use existing S3-compatible SDKs, tools, and command-line utilities without modifying your applications.

---

## Common Use Cases

- **Web Assets & Media**: Host images, audio, video files, documents, and static website content.
- **Backups & Long-Term Archives**: Store database dumps, system backups, logs, and audit trails securely and cost-effectively.
- **Big Data & Analytics**: Serve as a high-throughput data lake for data processing and machine learning workflows.
- **Application File Storage**: Allow users to directly upload and download files via pre-signed URLs.

---

## Key Features

- **100% S3 API Compatible**: Works seamlessly with AWS CLI (`aws s3`), `s3cmd`, `rclone`, Terraform, and AWS SDKs (Python `boto3`, Go, Node.js, Java, etc.).
- **High Durability & Availability**: Data is distributed and protected using Ceph erasure coding or replication across hardware nodes in your datacenter.
- **Project Multi-Tenancy**: Isolated storage buckets with dedicated access keys and granular access controls per project.
- **Automated Lifecycle Policies**: Automatically expire and delete temporary objects after a specified number of days to save space.

---

## Credentials & Access Configuration

### S3 Access Keys

To interact with Object Storage using standard S3 client tools or SDKs, generate project-scoped S3 credentials from **Storage** > **Object Storage** > **Access Keys**:

- **Access Key ID**: The public identifier for your API credentials.
- **Secret Access Key**: The private signature key used to authenticate API requests.
- **Endpoint URL**: The HTTPS entry point for your region/cluster (e.g., `https://s3.your-domain.net`).

### Bucket Provisioning

Buckets can be created directly from the Superphenix web console under **Storage** > **Object Storage** or dynamically via S3 API calls (`CreateBucket`). Bucket names must be globally unique across the object store and conform to standard DNS naming rules (lowercase letters, numbers, and hyphens).

---

### AWS CLI Integration

Configure the AWS CLI with your credentials:

```bash
aws configure
# AWS Access Key ID: <your-access-key-id>
# AWS Secret Access Key: <your-secret-access-key>
# Default region name: default
# Default output format: json
```

Use the `--endpoint-url` flag to interact with your Superphenix Object Storage:

```bash
# List all your buckets
aws --endpoint-url https://s3.your-domain.net s3 ls

# Upload a file
aws --endpoint-url https://s3.your-domain.net s3 cp photo.jpg s3://my-app-uploads/

# Download a file
aws --endpoint-url https://s3.your-domain.net s3 cp s3://my-app-uploads/photo.jpg ./downloaded-photo.jpg

# List files in the bucket
aws --endpoint-url https://s3.your-domain.net s3 ls s3://my-app-uploads/
```

---

### Python (boto3) Integration

Connecting from your applications is straightforward using standard S3 client libraries:

```python
import boto3

# Initialize S3 client
s3 = boto3.client(
    "s3",
    endpoint_url="https://s3.your-domain.net",
    aws_access_key_id="YOUR_ACCESS_KEY_ID",
    aws_secret_access_key="YOUR_SECRET_ACCESS_KEY",
)

# Upload an object
s3.upload_file("report.pdf", "my-app-uploads", "reports/2026-q3.pdf")

# Generate a temporary pre-signed URL for client downloads (valid for 1 hour)
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "my-app-uploads", "Key": "reports/2026-q3.pdf"},
    ExpiresIn=3600,
)
print(f"Download URL: {url}")
```

---

## Bucket Lifecycle Policies

Configure automated lifecycle rules to keep your storage organized and avoid unnecessary costs:

- **Expiration Rules**: Automatically delete objects in designated folders (or the entire bucket) after a set number of days (e.g., automatically delete files in `temp/` after 7 days).
- **Incomplete Multipart Upload Cleanup**: Automatically abort and purge unfinished multipart uploads after a specified period to reclaim space.

---

## Best Practices

- **Use Multipart Uploads for Large Files**: When uploading files larger than 100 MB, use multipart uploads. Standard tools like `aws s3 cp` and `boto3` do this automatically to ensure fast, resumable transfers.
- **Rotate Credentials Regularly**: Generate separate access keys for each application service, and rotate them periodically.
- **Leverage Pre-Signed URLs**: Instead of streaming large file downloads through your application servers, generate time-limited pre-signed URLs so clients can download directly from Object Storage.
