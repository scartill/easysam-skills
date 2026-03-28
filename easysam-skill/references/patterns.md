# EasySAM Resource Patterns

## Lambda + DynamoDB (with IAM)
```yaml
functions:
  my-function:
    uri: backend/handler.py
    tables:
      - !Ref MyTable
    envvars:
      TABLE_NAME: !Ref MyTable

tables:
  MyTable:
    attributes:
      - name: id
        hash: true
```

## SQS-Triggered Lambda
```yaml
queues:
  task-queue:

functions:
  worker:
    uri: backend/worker.py
    polls:
      - name: !Ref task-queue
        batchsize: 10
```

## Scheduled Cleanup
```yaml
functions:
  cleanup:
    uri: backend/cleanup.py
    schedule: "rate(1 day)"
```
