# Object Storage

Superphenix **Object Storage** provides S3-compatible object storage backed by Ceph RADOS Gateway (RGW). Buckets and objects are accessible over HTTPS using standard AWS S3 APIs, SDKs, and CLI tools.

---

## Architecture and Integration

- **S3 API Compatibility**: Compatible with the AWS CLI, Terraform S3 providers, `rclone`, `s3cmd`, and AWS SDKs (boto3, Go, Node.js, Java).
- **Data Protection**: Objects are stored across the underlying Ceph cluster using erasure coding or replication profiles defined at deployment.
- **Project Multi-Tenancy**: Storage buckets and credentials are scoped per project, ensuring tenant data isolation.

---

## Credentials and Endpoints

S3 credentials are generated in the web console under **Storage** > **Object Storage** > **Access Keys**:

- **Access Key ID**: Public identifier for the project API credential.
- **Secret Access Key**: Private signature key for signing requests.
- **Endpoint URL**: The HTTPS service URL (for example, `https://s3.your-domain.net`).

---

## Bucket Management

Buckets can be created in the web console under **Storage** > **Object Storage** or via standard S3 API calls (`CreateBucket`).

- **Naming Requirements**: Bucket names must be unique across the object storage cluster and adhere to DNS naming rules (lowercase alphanumeric characters, dots, and hyphens).
- **Region Name**: Standard S3 tools should specify `default` or the region name configured by your platform administrator.

---

## Client Configuration Examples

### AWS CLI

Configure credentials:

```bash
aws configure
# AWS Access Key ID: <your-access-key-id>
# AWS Secret Access Key: <your-secret-access-key>
# Default region name: default
# Default output format: json
```

Run S3 commands by passing the regional endpoint:

```bash
# List project buckets
aws --endpoint-url https://s3.your-domain.net s3 ls

# Upload an object
aws --endpoint-url https://s3.your-domain.net s3 cp sample.tar.gz s3://my-app-data/

# Download an object
aws --endpoint-url https://s3.your-domain.net s3 cp s3://my-app-data/sample.tar.gz ./sample.tar.gz

# List bucket contents
aws --endpoint-url https://s3.your-domain.net s3 ls s3://my-app-data/
```
