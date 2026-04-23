# Prismarine CRUD API

The generated `prismarine_client.py` contains model classes (e.g., `TeamModel`) with static methods for database operations.

## Common Operations

### `get(*, PK_val, SK_val=None, default=..., **kwargs) -> Model`
Retrieves a single item. Raises `DbNotFound` if not found and no default is provided.

### `put(item: Model, **kwargs) -> Model`
Creates or replaces an item.

### `update(dto: UpdateDTO, *, PK_val, SK_val=None, default=...) -> Model`
Updates an existing item. Merges the DTO with the existing item.

### `save(updated: Model, *, original: Model | None = None) -> Model`
Intelligently saves an item by calculating a diff and performing an `update_item` call.

### `delete(*, PK_val, SK_val=None, **kwargs)`
Deletes an item.

### `list(*, PK_val, **kwargs) -> List[Model]`
Queries items by partition key. Only available if a Sort Key (SK) is defined.

### `scan(**kwargs) -> List[Model]`
Scans the entire table.

## Secondary Index Operations

If an index named `by-data` is defined, the model class will have a nested class `ByData`:

### `ByData.list(*, PK_val, limit=None, direction='ASC') -> List[Model]`
Queries the index.

### `ByData.get(*, PK_val, SK_val) -> Model`
Retrieves a single item from the index (if SK is defined for the index).

## Usage Tips

- **Named Arguments**: Prismarine requires named arguments for most methods to prevent silent failures when schemas change.
- **DbNotFound**: Always handle `DbNotFound` from `prismarine.runtime` when using `get`.
- **Extension**: It is recommended to create a `db.py` that inherits from the generated model classes to add custom logic.
