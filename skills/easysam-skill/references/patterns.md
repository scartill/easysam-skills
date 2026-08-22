# EasySAM Resource Patterns

These patterns demonstrate correct EasySAM syntax for `resources.yaml`, `deploy-context.yaml`, and module-level `easysam.yaml` resource definitions.

## Global Project Configuration (`resources.yaml`)
*File: resources.yaml*
```yaml
prefix: my-app
python: 3.12
tags:
  Project: EasySAMApp
  Owner: DevOps
envvars:
  ENVIRONMENT: "{{environment}}"
import:
  - backend
```

## Environment Overrides (`deploy-context.yaml`)
*File: deploy-context.yaml*
```yaml
dev:
  envvars:
    LOG_LEVEL: DEBUG
    EXTERNAL_API_URL: "https://dev-api.example.com"
prod:
  envvars:
    LOG_LEVEL: INFO
    EXTERNAL_API_URL: "https://api.example.com"
  vpc:
    security_group_ids:
      - sg-0123456789abcdef0
    subnet_ids:
      - subnet-0123456789abcdef0
      - subnet-0fe23456789abcdef0
```

## Lambda + DynamoDB (with IAM)
*File: backend/database/easysam.yaml*
```yaml
tables:
  MyTable:
    attributes:
      - name: pk
        hash: true
      - name: sk
        range: true

lambda:
  name: db-worker
  resources:
    tables:
      - MyTable      # Correct: bare name, NO !Ref
    envvars:
      TABLE_NAME: MyTable
      API_KEY: "{{resolve:ssm:/myapp/api-key}}" # Correct: SSM resolve syntax
```

## DynamoDB Table with Global Secondary Index (GSI)
*File: backend/database/easysam.yaml*
```yaml
tables:
  UsersTable:
    attributes:
      - name: id
        hash: true
      - name: email
        type: S
    gsis:
      - name: EmailIndex
        hash: email
```

## HTTP-Facing Lambda
*File: backend/function/api/easysam.yaml*
```yaml
lambda:
  name: api-handler
  integration:       # Correct: use integration, not api
    path: /api/v1    # Correct: unique path prefix
    open: true
```

## SQS-Triggered Lambda
*File: backend/function/worker/easysam.yaml*
```yaml
queues:
  task-queue:

lambda:
  name: worker
  polls:
    - name: task-queue # Correct: bare queue name
      batchsize: 10
```

## SNS Topic & Subscription
*File: backend/notifications/easysam.yaml*
```yaml
topics:
  user-events:

lambda:
  name: event-processor
  subscribes:
    - name: user-events # Correct: bare topic name
```

## S3 Bucket (Public Shortcut)
*File: backend/storage/easysam.yaml*
```yaml
buckets:
  assets:
    public: true     # Correct: shortcut for public CORS
```

## Scheduled Lambda (Poller)
*File: backend/function/poller/easysam.yaml*
```yaml
lambda:
  name: poller
  schedule: "rate(5 minutes)"
  # NOTE: Poller has no integration: block
```
