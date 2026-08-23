1: # Model Definition in Prismarine
2: 
3: Prismarine uses the `Cluster` class to group and register DynamoDB models. Models can be defined using Python `TypedDict` (default) or `pydantic.BaseModel`.
4: 
5: > **MODELLING MODE REQUIREMENT**: Specify `modelling` explicitly in `resources.yaml` (`modelling: typed-dict` or `modelling: pydantic`). When using `typed-dict` mode, model classes MUST inherit from `TypedDict`. When using `pydantic` mode, model classes MUST inherit from `BaseModel` (and `pydantic` must be specified as a Lambda dependency).
6: 
7: ---
8: 
9: ## 1. Initializing the Cluster

Initialize a cluster in `common/<package>/models.py`:

```python
from prismarine.runtime import Cluster
c = Cluster('MyApp')  # Must start with EasySAM prefix
```

---

## 2. `@c.model` Decorator

Registers a class as a DynamoDB model table.

### Decorator Parameters:
- **`PK`** (required): Partition key attribute name (`str`).
- **`SK`** (optional): Sort key attribute name (`str`).
- **`table`** (optional): Custom exact table name (overrides cluster prefixing).
- **`name`** (optional): Custom model name (prepends cluster prefix).
- **`trigger`** (optional): EasySAM Lambda trigger configuration (`str` or `dict`).
- **`ttl`** (optional): Attribute name designated for DynamoDB TTL (`str`).

---

## 3. `@c.index` Decorator

Defines a Global Secondary Index (GSI).

> **CRITICAL RULE**: `@c.index(...)` decorators **MUST BE PLACED ABOVE** the `@c.model(...)` decorator.

### Decorator Parameters:
- **`index`** (required): Name of the index in DynamoDB (`str`).
- **`PK`** (required): Partition key attribute name for the index (`str`).
- **`SK`** (optional): Sort key attribute name for the index (`str`).

---

## 4. Code Examples

### A. TypedDict Mode (Default)
*File: common/orders/models.py*
```python
from typing import TypedDict, NotRequired
from prismarine.runtime import Cluster

c = Cluster('MyApp')

@c.index(index='by-customer', PK='CustomerId', SK='CreatedAt')  # ABOVE @c.model
@c.model(PK='OrderId', SK='ItemType', ttl='ExpireAt', trigger='order-events')
class OrderItem(TypedDict):
    OrderId: str
    ItemType: str
    CustomerId: str
    CreatedAt: str
    Quantity: int
    Price: float
    ExpireAt: NotRequired[int]
```

### B. Pydantic Mode (`modelling: pydantic`)
*File: common/orders/models.py*
```python
from pydantic import BaseModel
from typing import Optional
from prismarine.runtime import Cluster

c = Cluster('MyApp')

@c.index(index='by-customer', PK='CustomerId', SK='CreatedAt')
@c.model(PK='OrderId', SK='ItemType', ttl='ExpireAt')
class OrderItem(BaseModel):
    OrderId: str
    ItemType: str
    CustomerId: str
    CreatedAt: str
    Quantity: int
    Price: float
    ExpireAt: Optional[int] = None
```

---

## 5. `@c.export` Decorator

Exports nested data structures or custom types so they are included in generated client imports:

```python
@c.export
class Address(TypedDict):
    Street: str
    City: str
    Zip: str
```
