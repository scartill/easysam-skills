---
name: prismarine-skill
description: Build, define, and manage DynamoDB models and client code using Prismarine, the model-driven DynamoDB ORM for EasySAM and Python. Always use this skill when creating Prismarine clusters, defining @c.model or @c.index decorators, configuring resources.yaml prismarine settings, handling TypedDict or Pydantic modelling modes, or performing CRUD database operations via prismarine_client.
---

# Prismarine Skill

This skill provides guidelines and patterns for using **Prismarine**, a model-driven DynamoDB ORM for EasySAM and Python applications.

## Core Directives & Rules

1. **Models MUST Be in `models.py`**:
   Define Prismarine models inside a file explicitly named `models.py` (e.g., `common/myobject/models.py`). Do NOT define models in `__init__.py`.
2. **Cluster Prefix MUST Match EasySAM `prefix`**:
   The prefix passed to `Cluster('MyPrefix')` in `models.py` **must start with the master `prefix`** defined in `resources.yaml` (e.g., if `resources.yaml` prefix is `my-app`, cluster prefix must be `my-app` or `my-app-users`).
3. **Do NOT Manually Define DynamoDB Tables in `easysam.yaml`**:
   EasySAM automatically inspects Prismarine models during preprocessing to create and register DynamoDB tables, indexes, TTL, and stream triggers in CloudFormation.
4. **Order Decorators Correctly**:
   Place `@c.index(...)` decorators **ABOVE** `@c.model(...)` decorators.
5. **Do NOT Run Standalone `prismarine generate-client` Commands in EasySAM**:
   In EasySAM projects, Prismarine client code (`prismarine_client.py`) is **automatically generated as an integrated step of `easysam generate .` and `easysam deploy .`**. Do NOT run separate `prismarine generate-client` CLI commands.

---

## Standard Project Layout

When integrating Prismarine with EasySAM, use the following package structure:

```text
my-project/
├── resources.yaml            # Root config with prismarine: section
├── common/
│   ├── dynamo_access.py      # Access module for environment-suffixed tables
│   └── myobject/
│       ├── models.py         # Prismarine Cluster & model definitions
│       └── prismarine_client.py # Auto-generated client code (created by easysam generate/deploy)
├── backend/
│   └── function/
│       └── my-function/
│           ├── easysam.yaml  # Lambda function definition
│           └── index.py      # Handler importing common.myobject.prismarine_client
```

---

## Data Access Workflow

### 1. Define Model Cluster (`common/myobject/models.py`)
```python
from typing import TypedDict, NotRequired
from prismarine.runtime import Cluster

c = Cluster('MyApp')

@c.index(index='by-email', PK='Email')  # Must be ABOVE @c.model
@c.model(PK='Id', SK='Type', ttl='ExpireAt', trigger='itemlogger')
class UserRecord(TypedDict):
    Id: str
    Type: str
    Email: str
    Name: str
    ExpireAt: NotRequired[int]
```

### 2. Configure `resources.yaml`
```yaml
prefix: my-app
python: 3.12
prismarine:
  default-base: common
  access-module: common.dynamo_access
  modelling: typed-dict          # or: pydantic
  tables:
    - package: myobject
      trigger: true              # Preserve model-defined triggers
```

### 3. Generate & Deploy (Automatic)
Simply run standard EasySAM commands. EasySAM handles Prismarine table preprocessing and client generation internally:
```bash
# Preprocesses models, validates schema, generates template, and writes prismarine_client.py
uv run easysam --environment dev generate .

# Builds and deploys application and Prismarine models to AWS
uv run easysam --environment dev --aws-profile <profile> deploy .
```
*(Note: Standalone `prismarine generate-client` CLI usage is only for non-EasySAM standalone projects).*

### 4. Execute CRUD Operations
```python
from common.myobject.prismarine_client import UserRecordModel
from prismarine.runtime import DbNotFound

# Create / Replace
UserRecordModel.put({'Id': 'usr_123', 'Type': 'profile', 'Email': 'user@example.com', 'Name': 'Alice'})

# Get
try:
    user = UserRecordModel.get(Id='usr_123', Type='profile')
except DbNotFound:
    user = None

# Query Secondary Index
users = UserRecordModel.ByEmail.list(Email='user@example.com')
```

---

## Reference Material

- **[easysam.md](references/easysam.md)**: EasySAM integration syntax (`resources.yaml`, stream triggers, conditional tables, TTL).
- **[model-definition.md](references/model-definition.md)**: Complete `@c.model`, `@c.index`, `@c.export` API & Pydantic mode.
- **[crud-api.md](references/crud-api.md)**: Generated client methods (`get`, `put`, `update`, `save`, `delete`, `list`, `scan`).
- **[cli-usage.md](references/cli-usage.md)**: Standalone Prismarine CLI commands (non-EasySAM projects only).
