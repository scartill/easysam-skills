# Prismarine Skill Feedback

**Date:** 2026-08-23
**Context:** Discovered gaps while adding `DbConditionFailed` to prismarine, setting up dynamo access, and fixing dedup.py to use the generated client properly.

---

## 1. Add "DynamoDB Access Module" Section

The skill should explain that environment-suffixed tables require a custom dynamo access module. Without this, agents will construct table names manually and call internal `_put_item` directly.

### Suggested content:

When deploying multi-environment stacks, DynamoDB tables are suffixed with the environment/stage name (e.g., `ScarBotMessageDedup-scarbotprod`). The generated prismarine client uses a **dynamo access module** to resolve actual table names at runtime.

**Always configure `access-module` in `resources.yaml`** when tables are environment-suffixed:

```yaml
prismarine:
  default-base: common
  access-module: common.dynamo_access
  modelling: typed-dict
  tables:
    - package: orm
```

**Create the access module** (`common/dynamo_access.py`):

```python
import os
import boto3
from prismarine.runtime.dynamo_access import DynamoAccess

DYNAMO = boto3.resource('dynamodb')


class MyDynamoAccess(DynamoAccess):
    def get_resource(self):
        return DYNAMO

    def get_table(self, full_model_name: str):
        env = os.environ.get('ENV', 'dev')
        return self.get_resource().Table(f'{full_model_name}-{env}')


dynamoaccess = MyDynamoAccess()


def get_dynamo_access():
    return dynamoaccess
```

**Key rules:**

- The access module MUST export a `get_dynamo_access()` function returning a `DynamoAccess` subclass.
- `get_table()` receives the logical table name (e.g., `ScarBotMessageDedup`) and must return the boto3 `Table` object for the actual deployed name.
- Without `access-module`, the generated client uses `prismarine.runtime.dynamo_default` which does NO name transformation — the logical name must match the deployed table name exactly.
- Consumer code NEVER constructs table names manually. Use the generated `XxxModel.put()`, `.get()`, etc. — the access module handles name resolution transparently.

---

## 2. Add "Conditional Writes & DbConditionFailed" Section

The skill doesn't mention conditional writes or the `DbConditionFailed` exception. Agents won't know this is available.

### Suggested content:

The generated `put()` method passes `**kwargs` through to DynamoDB's `put_item`. Use `ConditionExpression` for atomic operations:

```python
from common.orm.prismarine_client import MessageDedupModel
from prismarine.runtime import DbConditionFailed

try:
    MessageDedupModel.put(
        {'MessageFingerprint': fingerprint, 'ExpireAt': expire_at},
        ConditionExpression='attribute_not_exists(MessageFingerprint)',
    )
except DbConditionFailed:
    # Item already exists — condition failed
    pass
```

`DbConditionFailed` is raised by prismarine's `_put_item` when DynamoDB returns `ConditionalCheckFailedException`. Import it from `prismarine.runtime`.

---

## 3. Clarify Modelling Mode Requirement

We hit a `NameError: name 'BaseModel' is not defined` because the model used Pydantic `BaseModel` but the generated client was in typed-dict mode (no `BaseModel` import).

### Suggested rule addition:

> **Specify `modelling` explicitly in `resources.yaml`.**
> Always set `modelling: typed-dict` or `modelling: pydantic`. When using `typed-dict` mode, models MUST inherit from `TypedDict`. When using `pydantic` mode, models MUST inherit from `BaseModel` and `pydantic` must be a Lambda dependency.

---

## Summary

These three additions would prevent:

- Using raw `_put_item` / `get_dynamo_access` instead of the generated client
- Constructing table names manually (bypassing the access layer)
- Not knowing `DbConditionFailed` exists or how to use conditional writes
- Mismatching model base class with the configured modelling mode
