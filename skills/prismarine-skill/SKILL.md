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
20: 5. **Do NOT Run Standalone `prismarine generate-client` Commands in EasySAM**:
21:    In EasySAM projects, Prismarine client code (`prismarine_client.py`) is **automatically generated as an integrated step of `easysam generate .` and `easysam deploy .`**. Do NOT run separate `prismarine generate-client` CLI commands.
22: 6. **Configure `access-module` for Environment-Suffixed Tables**:
23:    When deploying multi-environment stacks, tables are suffixed with stage/environment names. Configure `access-module: common.dynamo_access` in `resources.yaml` and create a `DynamoAccess` module exporting `get_dynamo_access()`. Consumer code MUST NEVER construct table names manually or call low-level `_put_item` directly—always use generated model methods (`Model.put()`, `Model.get()`).
24: 7. **Use `DbConditionFailed` for Conditional Writes**:
25:    Pass `ConditionExpression` to `put()` for atomic conditional writes. Import `DbConditionFailed` from `prismarine.runtime` to catch `ConditionalCheckFailedException`.
26: 8. **Match `modelling` Mode with Model Parent Class**:
27:    Explicitly set `modelling: typed-dict` or `modelling: pydantic` in `resources.yaml`. Models MUST inherit from `TypedDict` when using `typed-dict` mode, or `BaseModel` when using `pydantic` mode.
28: 
29: ---
30: 
31: ## Standard Project Layout
32: 
33: When integrating Prismarine with EasySAM, use the following package structure:
34: 
35: ```text
36: my-project/
37: ├── resources.yaml            # Root config with prismarine: section
38: ├── common/
39: │   ├── dynamo_access.py      # Access module for environment-suffixed tables
40: │   └── myobject/
41: │       ├── models.py         # Prismarine Cluster & model definitions
42: │       └── prismarine_client.py # Auto-generated client code (created by easysam generate/deploy)
43: ├── backend/
44: │   └── function/
45: │       └── my-function/
46: │           ├── easysam.yaml  # Lambda function definition
47: │           └── index.py      # Handler importing common.myobject.prismarine_client
48: ```
49: 
50: ---
51: 
52: ## Data Access Workflow
53: 
54: ### 1. Define Model Cluster (`common/myobject/models.py`)
55: ```python
56: from typing import TypedDict, NotRequired
57: from prismarine.runtime import Cluster
58: 
59: c = Cluster('MyApp')
60: 
61: @c.index(index='by-email', PK='Email')  # Must be ABOVE @c.model
62: @c.model(PK='Id', SK='Type', ttl='ExpireAt', trigger='itemlogger')
63: class UserRecord(TypedDict):
64:     Id: str
65:     Type: str
66:     Email: str
67:     Name: str
68:     ExpireAt: NotRequired[int]
69: ```
70: 
71: ### 2. Configure `resources.yaml`
72: ```yaml
73: prefix: my-app
74: python: 3.12
75: prismarine:
76:   default-base: common
77:   access-module: common.dynamo_access
78:   modelling: typed-dict          # or: pydantic
79:   tables:
80:     - package: myobject
81:       trigger: true              # Preserve model-defined triggers
82: ```
83: 
84: ### 3. Generate & Deploy (Automatic)
85: Simply run standard EasySAM commands. EasySAM handles Prismarine table preprocessing and client generation internally:
86: ```bash
87: # Preprocesses models, validates schema, generates template, and writes prismarine_client.py
88: uv run easysam --environment dev generate .
89: 
90: # Builds and deploys application and Prismarine models to AWS
91: uv run easysam --environment dev --aws-profile <profile> deploy .
92: ```
93: *(Note: Standalone `prismarine generate-client` CLI usage is only for non-EasySAM standalone projects).*
94: 
95: ### 4. Execute CRUD Operations
96: ```python
97: from common.myobject.prismarine_client import UserRecordModel
98: from prismarine.runtime import DbNotFound, DbConditionFailed
99: 
100: # Create / Replace
101: UserRecordModel.put({'Id': 'usr_123', 'Type': 'profile', 'Email': 'user@example.com', 'Name': 'Alice'})
102: 
103: # Conditional Write
104: try:
105:     UserRecordModel.put(
106:         {'Id': 'usr_123', 'Type': 'profile', 'Email': 'user@example.com', 'Name': 'Alice'},
107:         ConditionExpression='attribute_not_exists(Id)',
108:     )
109: except DbConditionFailed:
110:     pass  # Item already exists
111: 
112: # Get
113: try:
114:     user = UserRecordModel.get(Id='usr_123', Type='profile')
115: except DbNotFound:
116:     user = None
117: 
118: # Query Secondary Index
119: users = UserRecordModel.ByEmail.list(Email='user@example.com')
120: ```

---

## Reference Material

- **[easysam.md](references/easysam.md)**: EasySAM integration syntax (`resources.yaml`, stream triggers, conditional tables, TTL).
- **[model-definition.md](references/model-definition.md)**: Complete `@c.model`, `@c.index`, `@c.export` API & Pydantic mode.
- **[crud-api.md](references/crud-api.md)**: Generated client methods (`get`, `put`, `update`, `save`, `delete`, `list`, `scan`).
- **[cli-usage.md](references/cli-usage.md)**: Standalone Prismarine CLI commands (non-EasySAM projects only).
