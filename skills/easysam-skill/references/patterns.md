# EasySAM Resource Patterns & Supported Resource Types

This document provides comprehensive YAML patterns for all supported AWS resource types and features in EasySAM.

---

## Supported Top-Level Resource Types

EasySAM supports 14 top-level resource and configuration keys in `resources.yaml` or module-level `easysam.yaml`:

| Top-Level Key | Description | Example Pattern Section |
| --- | --- | --- |
| `prefix` | Global project resource name prefix (**required**). | Global Configuration |
| `python` | Python runtime version (`"3.12"`, `"3.13"`, `"3.14"`). | Global Configuration |
| `tags` | AWS tags applied across generated resources. | Global Configuration |
| `envvars` | Global environment variables & SSM resolvers. | Global Configuration |
| `import` | Modular sub-directory imports. | Global Configuration |
| `lambda` / `functions` | Lambda function compute definitions. | Compute Patterns |
| `tables` | DynamoDB tables (keys, GSIs, TTL, stream triggers). | Database Patterns |
| `buckets` | S3 storage buckets (`public: true`, access policies). | Storage Patterns |
| `queues` | SQS message queues. | Messaging Patterns |
| `topics` | SNS notification topics. | Messaging Patterns |
| `streams` | Kinesis Data Streams delivering into S3. | Analytics Patterns |
| `search` | OpenSearch Serverless (AOSS) collections. | Search Patterns |
| `authorizers` | API Gateway custom Lambda authorizers. | Security Patterns |
| `mqtt` | AWS IoT MQTT custom authorizers & topic rules. | IoT Patterns |
| `prismarine` | Model-driven DynamoDB ORM integration. | Prismarine Patterns |
| `plugins` | Custom Jinja2 template extension plugins. | Plugin Patterns |

---

## 1. Global & Environment Patterns

### Global Configuration (`resources.yaml`)
```yaml
prefix: my-app
python: "3.14"    # Supported: "3.12", "3.13", "3.14"
tags:
  Project: EasySAMApp
  Owner: DevOps
envvars:
  ENVIRONMENT: "{{environment}}"
import:
  - backend
```

### Conditional Overrides (`deploy-context.yaml`)
```yaml
dev:
  envvars:
    LOG_LEVEL: DEBUG
prod:
  envvars:
    LOG_LEVEL: INFO
  vpc:
    security_group_ids:
      - sg-0123456789abcdef0
    subnet_ids:
      - subnet-0123456789abcdef0
```

---

## 2. Compute Patterns (`lambda:`)

### HTTP API Gateway (FastAPI / Greedy Route)
```yaml
lambda:
  name: api-handler
  integration:
    path: /api/v1
    greedy: true      # Mandatory for FastAPI sub-routing
    open: true
```

### AWS Lambda Function URL
```yaml
lambda:
  name: direct-url-func
  function_url:
    auth_type: NONE   # 'NONE' or 'AWS_IAM'
    invoke_mode: BUFFERED # 'BUFFERED' or 'RESPONSE_STREAM'
```

### Lambda Custom Layers
```yaml
lambda:
  name: custom-layered-func
  layers:
    - "arn:aws:lambda:us-east-1:123456789012:layer:FFmpeg:1"
```

### Scheduled Lambda (EventBridge Cron/Rate)
```yaml
lambda:
  name: cron-job
  schedule: "rate(10 minutes)" # Or cron expression: "cron(0 12 * * ? *)"
```

---

## 3. Storage, Database & Search Patterns

### OpenSearch Serverless (AOSS Vector Search)
```yaml
search:
  vector-index:
    type: vectorsearch # OpenSearch Serverless collection
```

### Kinesis Data Stream into S3
```yaml
streams:
  telemetry-stream:
    buckets:
      raw-bucket:
        bucketprefix: telemetry/
        intervalinseconds: 300
```

### DynamoDB Table with GSI & TTL
```yaml
tables:
  UsersTable:
    attributes:
      - name: id
        hash: true
      - name: email
        type: S
    indices:
      - name: EmailIndex
        attributes:
          - name: email
            hash: true
    ttl: ExpireAt
```

### S3 Bucket (Public Shortcut)
```yaml
buckets:
  assets:
    public: true
```

---

## 4. Messaging & Security Patterns

### SQS Poller Lambda
```yaml
queues:
  task-queue:

lambda:
  name: worker
  polls:
    - name: task-queue
      batchsize: 10
```

### SNS Topic Subscriber Lambda
```yaml
topics:
  user-events:

lambda:
  name: notifier
  subscribes:
    - name: user-events
```

### API Gateway Custom Authorizer
```yaml
authorizers:
  jwt-auth:
    function: auth-lambda
    identity_source: method.request.header.Authorization
    ttl: 300
```

### AWS IoT MQTT Authorizer
```yaml
mqtt:
  authorizer:
    function: iot-auth-lambda
  topics:
    - "telemetry/#"
```
