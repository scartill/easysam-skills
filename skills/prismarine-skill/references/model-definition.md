# Model Definition in Prismarine

Prismarine uses the `Cluster` class to group and define models. Models can be defined using Python's `TypedDict` (default) or `pydantic.BaseModel`.

## The Cluster Class

Initialize a cluster with an optional prefix:

```python
from prismarine.runtime import Cluster
c = Cluster('MyPrefix')
```

## @c.model Decorator

Registers a class as a DynamoDB model.

### Parameters:
- **`PK`** (required): The name of the partition key attribute.
- **`SK`** (optional): The name of the sort key attribute.
- **`table`** (optional): Full custom table name (ignores prefix).
- **`name`** (optional): Custom model name (prepends prefix).
- **`trigger`** (optional): EasySAM Lambda trigger configuration.
- **`ttl`** (optional): Attribute name for DynamoDB TTL.

### Example:
```python
@c.model(PK='UserId', SK='RecordId')
class UserRecord(TypedDict):
    UserId: str
    RecordId: str
    Data: str
```

## @c.index Decorator

Defines a Secondary Index. **Must be placed above `@c.model`**.

### Parameters:
- **`index`** (required): Name of the index in DynamoDB.
- **`PK`** (required): Partition key for the index.
- **`SK`** (optional): Sort key for the index.

### Example:
```python
@c.index(index='by-data', PK='Data')
@c.model(PK='UserId', SK='RecordId')
class UserRecord(TypedDict):
    ...
```

## @c.export Decorator

Exports a class (e.g., a nested TypedDict) that is not a model but is used as a type within models.

```python
@c.export
class Metadata(TypedDict):
    Created: str
```
