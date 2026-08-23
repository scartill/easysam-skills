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

23: ### `put(item: Model, **kwargs) -> Model`
24: Creates or replaces an item in DynamoDB. Extra `kwargs` are passed directly through to DynamoDB `put_item` (e.g., `ConditionExpression`).
25: 
26: ```python
27: from prismarine.runtime import DbConditionFailed
28: 
29: # Basic Put
30: UserRecordModel.put({
31:     'Id': 'usr_100',
32:     'Type': 'profile',
33:     'Email': 'user@example.com',
34:     'Name': 'Alice'
35: })
36: 
37: # Conditional Write (raises DbConditionFailed if condition is not met)
38: try:
39:     UserRecordModel.put(
40:         {'Id': 'usr_100', 'Type': 'profile', 'Email': 'user@example.com', 'Name': 'Alice'},
41:         ConditionExpression='attribute_not_exists(Id)',
42:     )
43: except DbConditionFailed:
44:     pass  # Item already exists
45: ```
46: 
47: ### `update(dto: UpdateDTO, *, PK_val, SK_val=None, default=...) -> Model`
48: Updates an existing item by merging non-null fields from DTO into the existing record.
49: 
50: ```python
51: UserRecordModel.update({'Name': 'Alice Smith'}, Id='usr_100', Type='profile')
52: ```
53: 
54: ### `save(updated: Model, *, original: Model | None = None) -> Model`
55: Diffs `updated` against `original` (or fetches current state) and executes a minimal `update_item` operation.
56: 
57: ```python
58: user['Name'] = 'Alice Johnson'
59: UserRecordModel.save(user)
60: ```
61: 
62: ### `delete(*, PK_val, SK_val=None, **kwargs)`
63: Deletes an item by partition key (and optional sort key).
64: 
65: ```python
66: UserRecordModel.delete(Id='usr_100', Type='profile')
67: ```
68: 
69: ### `list(*, PK_val, limit=None, direction='ASC', **kwargs) -> list[Model]`
70: Queries items matching partition key (available when a Sort Key is defined).
71: 
72: ```python
73: items = UserRecordModel.list(Id='usr_100', limit=20, direction='DESC')
74: ```
75: 
76: ### `scan(**kwargs) -> list[Model]`
77: Scans the entire DynamoDB table.
78: 
79: ---
80: 
81: ## 2. Secondary Index Queries
82: 
83: For each Global Secondary Index (e.g. `@c.index(index='by-email', PK='Email')`), a nested class (e.g. `ByEmail`) is generated:
84: 
85: ### `ByEmail.list(*, PK_val, limit=None, direction='ASC') -> list[Model]`
86: Queries items from the index by index partition key.
87: 
88: ```python
89: users = UserRecordModel.ByEmail.list(Email='user@example.com')
90: ```
91: 
92: ### `ByEmail.get(*, PK_val, SK_val=...) -> Model`
93: Retrieves a single item from the index (available if the index specifies a sort key).
94: 
95: ---
96: 
97: ## 3. Best Practices & Guidelines
98: 
99: 1. **Keyword Arguments**: Always pass named keyword arguments (e.g. `Id='usr_100'`) to prevent silent argument misalignment when keys or schemas evolve.
100: 2. **Exception Handling**: Always import `DbNotFound` (for `get()`) and `DbConditionFailed` (for conditional `put()`) from `prismarine.runtime`.
101: 3. **Pydantic Validation**: When `modelling: pydantic` is used in `resources.yaml`, `get()`, `put()`, and `list()` automatically construct and return Pydantic instances.
102: 4. **No Direct `_put_item` / Raw Calls**: Always use the generated client methods (`Model.put()`, `.get()`, etc.) rather than calling low-level `_put_item` directly.
