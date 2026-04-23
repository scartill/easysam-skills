# Prismarine & EasySAM Integration

Prismarine integrates with EasySAM to handle DynamoDB table creation and configuration in AWS.

## Automatic Table Generation

DynamoDB tables are **auto-generated** from your Prismarine models. **Do NOT** define these tables manually in your `easysam.yaml` files.

### Configuration (`resources.yaml`)

Configure the `prismarine:` section in your root `resources.yaml`:

```yaml
prismarine:
  default-base: common  # The base package for your models
  tables:
    - package: models   # Looks for common/models.py
```

## Lambda Triggers (DynamoDB Streams)

Configure Lambda triggers directly in the `@c.model` decorator in `models.py`.

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
        'batchsize': 10
    }
)
```

## Time To Live (TTL)

Configure TTL by specifying the attribute name in the model decorator.

```python
@c.model(PK='Id', ttl='ExpireAt')
class MyModel(TypedDict):
    Id: str
    ExpireAt: int  # Unix timestamp in seconds
```
