# Prismarine & EasySAM Integration

Prismarine seamlessly integrates with EasySAM to automate DynamoDB table creation, GSI indexing, TTL, stream triggers, and type-safe client generation.

---

## 1. EasySAM Configuration (`resources.yaml`)

Configure Prismarine settings under the top-level `prismarine:` key in `resources.yaml`:

```yaml
prefix: my-app
python: 3.12

prismarine:
  default-base: common                      # Default directory containing model packages
  access-module: common.dynamo_access      # Python module providing DynamoDB connection layer
  modelling: typed-dict                      # Modelling mode: 'typed-dict' (default) or 'pydantic'
  extra-imports:
    - common.myobject.models:NestedStruct   # Additional classes to import (module:Class)
  tables:
    - package: myobject                      # Package directory under base (common/myobject)
      base: common                           # Optional base override
      trigger: true                          # Preserve model-defined stream triggers
  conditional-tables:                        # Environment-conditional table packages
    ? !Conditional
      key: prod-tables
      environment: prod
    :
      - package: prodobject
```

### Options Summary

| Option | Type | Description |
| --- | --- | --- |
| `default-base` | `str` | Default root directory containing model packages (e.g. `common`). |
| `access-module` | `str` | Python module for runtime DynamoDB table resolution (default: `prismarine.runtime.dynamo_default`). |
| `modelling` | `str` | Model representation: `typed-dict` or `pydantic`. |
| `extra-imports` | `list[str]` | Extra imports for generated client (`module:ClassName`). |
| `tables` | `list[dict]` | List of table package definitions (`package`, `base`, `trigger`). |
| `conditional-tables` | `dict` | Environment/region-conditional table packages resolved via `!Conditional`. |

---

## 2. Two-Stage Build Pipeline

EasySAM executes Prismarine processing in two stages:

1. **Stage 1 (Preprocessing)**: During `easysam generate`/`deploy` loading:
   - Evaluates `conditional-tables` and appends matches to `tables`.
   - Imports `models.py` from each table package.
   - Validates that `Cluster` prefix starts with the EasySAM `prefix`.
   - Generates AWS CloudFormation `AWS::DynamoDB::Table` definitions (including GSIs, stream views, and TTL) and merges them into `resources.yaml` table definitions.
   - Strips stream triggers unless `trigger: true` is explicitly specified on the table entry.

2. **Stage 2 (Client Generation)**: After template rendering:
   - Builds `prismarine_client.py` for each package.
   - Writes the generated client to the target package directory (`common/<package>/prismarine_client.py`).

---

## 3. Mandatory Prefix Validation Rule

When initializing a cluster in `models.py`:

```python
from prismarine.runtime import Cluster
c = Cluster('my-app-db')
```

The cluster prefix (`'my-app-db'`) **MUST start with the master `prefix`** defined in `resources.yaml` (`prefix: my-app`). If the cluster prefix does not start with the EasySAM master prefix, schema validation fails.

---

## 4. Lambda Stream Triggers

Define stream triggers directly in the `@c.model` decorator in `models.py`:

```python
# Simple trigger (function name)
@c.model(PK='Id', trigger='item-logger-func')

# Advanced trigger configuration
@c.model(
    PK='Id',
    trigger={
        'function': 'item-logger-func',
        'viewtype': 'new-and-old',
        'batchsize': 10
    }
)
```

*Note*: Ensure `trigger: true` is set under the package entry in `resources.yaml` for the trigger to be active in the generated SAM template.

---

## 5. Time To Live (TTL)

Configure TTL by passing the attribute name to `ttl=` in `@c.model`:

```python
@c.model(PK='Id', ttl='ExpireAt')
class CacheItem(TypedDict):
    Id: str
    ExpireAt: int  # Unix timestamp in seconds
```

---

## 6. Access Module Implementation

Create `common/dynamo_access.py` to resolve environment-suffixed table names at runtime:

```python
import os
import boto3
from prismarine.runtime.dynamo_default import DynamoAccess

class AppDynamoAccess(DynamoAccess):
    def get_resource(self):
        return boto3.resource('dynamodb')

    def get_table(self, full_model_name: str):
        env = os.environ.get('ENVIRONMENT', 'dev')
        return self.get_resource().Table(f'{full_model_name}-{env}')
```
