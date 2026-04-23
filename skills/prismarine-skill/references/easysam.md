# Prismarine & EasySAM Integration

Prismarine integrates with EasySAM to handle DynamoDB table creation and configuration in AWS.

## Table Discovery

EasySAM can use `prismarine.prisma_easysam.build_dynamo_tables` to generate SAM table definitions from Prismarine clusters.

## Triggers (DynamoDB Streams)

Configure Lambda triggers directly in the `@c.model` decorator.

### Simple Trigger:
```python
@c.model(PK='Id', trigger='my-lambda-function')
```

### Advanced Trigger:
```python
@c.model(
    PK='Id',
    trigger={
        'function': 'my-lambda',
        'viewtype': 'new-and-old',
        'batchsize': 10,
        'batchwindow': 5,
        'startingposition': 'latest'
    }
)
```

## Time To Live (TTL)

Configure TTL by specifying the attribute name.

```python
@c.model(PK='Id', ttl='ExpireAt')
class MyModel(TypedDict):
    Id: str
    ExpireAt: int  # Unix timestamp in seconds
```
