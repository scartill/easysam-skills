# EasySAM Resource Patterns

These patterns should be placed within an `easysam.yaml` file in a specific module (e.g., `backend/database/easysam.yaml` or `backend/function/myfunc/easysam.yaml`).

## Lambda + DynamoDB (with IAM)
*File: backend/function/myfunc/easysam.yaml*
```yaml
lambda:
  name: my-function
  resources:
    tables:
      - !Ref MyTable
  envvars:
    TABLE_NAME: !Ref MyTable
```

*File: backend/database/easysam.yaml*
```yaml
tables:
  MyTable:
    attributes:
      - name: id
        hash: true
```

## SQS-Triggered Lambda
*File: backend/function/worker/easysam.yaml*
```yaml
queues:
  task-queue:

lambda:
  name: worker
  polls:
    - name: !Ref task-queue
      batchsize: 10
```

## Scheduled Cleanup
*File: backend/function/cleanup/easysam.yaml*
```yaml
lambda:
  name: cleanup
  schedule: "rate(1 day)"
```
