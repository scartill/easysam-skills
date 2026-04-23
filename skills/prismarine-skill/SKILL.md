---
name: prismarine-skill
description: Development guide for using the Prismarine DynamoDB ORM. Use when defining DynamoDB models, generating client code, or performing database operations using Prismarine.
---

# Prismarine Development Skill

This skill provides guidance for using Prismarine, a Pythonic ORM for DynamoDB.

## Core Workflow

1.  **Define Models**: Create a `models.py` file using `Cluster` and `@c.model` decorators.
2.  **Generate Client**: Use the `prismarine generate-client` CLI command.
3.  **Extend/Use**: Create a `db.py` file to extend the generated client and use it for CRUD operations.

## Model Definition

Models define the schema and keys for DynamoDB tables.

- Use `@c.model(PK='...', SK='...')` to define primary keys.
- Use `@c.index(...)` for Secondary Indexes (place ABOVE `@c.model`).
- See [model-definition.md](references/model-definition.md) for detailed decorator options.

## Client Generation

Generate the type-safe client using the CLI:

```bash
prismarine generate-client --base . myapp.models
```

- See [cli-usage.md](references/cli-usage.md) for all options (including Pydantic support).

## CRUD Operations

The generated client provides a high-level API for interacting with DynamoDB.

```python
from myapp.db import TeamModel

# Create
TeamModel.put({'Foo': 'val1', 'Bar': 'val2'})

# Get
item = TeamModel.get(foo='val1', bar='val2')

# Query (if SK exists)
items = TeamModel.list(foo='val1')
```

- See [crud-api.md](references/crud-api.md) for the complete API reference.

## EasySAM Integration

Prismarine models can include configuration for EasySAM, such as Lambda triggers and TTL.

- See [easysam.md](references/easysam.md) for integration details.
