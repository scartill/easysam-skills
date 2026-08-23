# Prismarine CRUD API Reference

The generated `prismarine_client.py` contains typed model classes (e.g. `UserRecordModel`, `OrderItemModel`) providing high-level static methods for DynamoDB operations.

---

## 1. Core Database Operations

### `get(*, PK_val, SK_val=None, default=..., **kwargs) -> Model`
Retrieves a single item by partition key (and optional sort key).
- Raises `DbNotFound` from `prismarine.runtime` if the item does not exist and no `default` is specified.

```python
from common.myobject.prismarine_client import UserRecordModel
from prismarine.runtime import DbNotFound

try:
    user = UserRecordModel.get(Id='usr_100', Type='profile')
except DbNotFound:
    user = None
```

### `put(item: Model, **kwargs) -> Model`
Creates or replaces an item in DynamoDB. Extra `kwargs` are passed directly through to DynamoDB `put_item` (e.g., `ConditionExpression`).

```python
from prismarine.runtime import DbConditionFailed

# Basic Put
UserRecordModel.put({
    'Id': 'usr_100',
    'Type': 'profile',
    'Email': 'user@example.com',
    'Name': 'Alice'
})

# Conditional Write (raises DbConditionFailed if condition is not met)
try:
    UserRecordModel.put(
        {'Id': 'usr_100', 'Type': 'profile', 'Email': 'user@example.com', 'Name': 'Alice'},
        ConditionExpression='attribute_not_exists(Id)',
    )
except DbConditionFailed:
    pass  # Item already exists
```

### `update(dto: UpdateDTO, *, PK_val, SK_val=None, default=...) -> Model`
Updates an existing item by merging non-null fields from DTO into the existing record.

```python
UserRecordModel.update({'Name': 'Alice Smith'}, Id='usr_100', Type='profile')
```

### `save(updated: Model, *, original: Model | None = None) -> Model`
Diffs `updated` against `original` (or fetches current state) and executes a minimal `update_item` operation.

```python
user['Name'] = 'Alice Johnson'
UserRecordModel.save(user)
```

### `delete(*, PK_val, SK_val=None, **kwargs)`
Deletes an item by partition key (and optional sort key).

```python
UserRecordModel.delete(Id='usr_100', Type='profile')
```

### `list(*, PK_val, limit=None, direction='ASC', **kwargs) -> list[Model]`
Queries items matching partition key (available when a Sort Key is defined).

```python
items = UserRecordModel.list(Id='usr_100', limit=20, direction='DESC')
```

### `scan(**kwargs) -> list[Model]`
Scans the entire DynamoDB table.

---

## 2. Secondary Index Queries

For each Global Secondary Index (e.g. `@c.index(index='by-email', PK='Email')`), a nested class (e.g. `ByEmail`) is generated:

### `ByEmail.list(*, PK_val, limit=None, direction='ASC') -> list[Model]`
Queries items from the index by index partition key.

```python
users = UserRecordModel.ByEmail.list(Email='user@example.com')
```

### `ByEmail.get(*, PK_val, SK_val=...) -> Model`
Retrieves a single item from the index (available if the index specifies a sort key).

---

## 3. Best Practices & Guidelines

1. **Keyword Arguments**: Always pass named keyword arguments (e.g. `Id='usr_100'`) to prevent silent argument misalignment when keys or schemas evolve.
2. **Exception Handling**: Always import `DbNotFound` (for `get()`) and `DbConditionFailed` (for conditional `put()`) from `prismarine.runtime`.
3. **Pydantic Validation**: When `modelling: pydantic` is used in `resources.yaml`, `get()`, `put()`, and `list()` automatically construct and return Pydantic instances.
4. **No Direct `_put_item` / Raw Calls**: Always use the generated client methods (`Model.put()`, `.get()`, etc.) rather than calling low-level `_put_item` directly.
