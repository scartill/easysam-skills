# EasySAM Resource Patterns

These patterns demonstrate correct EasySAM syntax for `resources.yaml`, `deploy-context.yaml`, module-level `easysam.yaml` resource definitions, and data access layers.

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

## Recommended Git Configuration (`.gitignore`)
*File: .gitignore*
```gitignore
# Python & Bytecode
__pycache__/
*.pyc

# Virtual Environment & Tooling
.venv/
.easysam/
template.yml
template.yaml

# EasySAM Generated Module Artifacts
**/common/
**/prismarine_clients/
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

## HTTP-Facing Lambda with FastAPI (Greedy Route)
*File: backend/function/api/easysam.yaml*
```yaml
lambda:
  name: api-handler
  integration:       # Use integration, not api
    path: /api/v1    # Base path prefix
    greedy: true     # MANDATORY for FastAPI to receive sub-path routes (/api/v1/*)
    open: true
```

## Data Access Pattern 1: Prismarine (Prisma for DynamoDB)
*File: backend/database/schema.prisma*
```prisma
datasource db {
  provider = "dynamodb"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prismarine-client-py"
  output   = "../../prismarine_clients/main"
}

model User {
  id        String   @id
  email     String   @unique
  createdAt DateTime @default(now())
}
```

## Data Access Pattern 2: Custom `DynamoAccess` (boto3 Helper)
*File: common/dynamo_access.py*
```python
import os
import boto3
from typing import Any

class DynamoAccess:
    """Lightweight boto3 wrapper for low-latency DynamoDB operations."""

    def __init__(self, table_name_env: str):
        table_name = os.environ[table_name_env]
        dynamodb = boto3.resource('dynamodb')
        self.table = dynamodb.Table(table_name)

    def get(self, pk: str, sk: str | None = None) -> dict[str, Any] | None:
        key = {'pk': pk}
        if sk:
            key['sk'] = sk
        res = self.table.get_item(Key=key)
        return res.get('Item')

    def put(self, item: dict[str, Any]) -> None:
        self.table.put_item(Item=item)
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
    public: true     # Shortcut for public CORS
```

## Scheduled Lambda (Poller)
*File: backend/function/poller/easysam.yaml*
```yaml
lambda:
  name: poller
  schedule: "rate(5 minutes)"
  # NOTE: Poller has no integration: block
```
