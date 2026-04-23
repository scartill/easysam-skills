# EasySAM Resource Patterns

These patterns should be placed within an `easysam.yaml` file in a specific module (e.g., `backend/database/easysam.yaml` or `backend/function/myfunc/easysam.yaml`).

## Lambda + DynamoDB (with IAM)
*File: backend/function/myfunc/easysam.yaml*
```yaml
lambda:
  name: my-function
  resources:
    tables:
      - MyTable      # Correct: use bare name
    envvars:
      TABLE_NAME: MyTable
      API_KEY: "{{resolve:ssm:/myapp/api-key}}" # Correct: resolve syntax
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
    - name: task-queue # Correct: use bare name
      batchsize: 10
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
