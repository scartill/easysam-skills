# Prismarine CLI Usage

The Prismarine CLI is used to generate the client code from your models.

## generate-client Command

Generates a `prismarine_client.py` file.

### Arguments:
- **`CLUSTER_PACKAGE`**: The Python package containing your models (e.g., `myapp.models`).

### Options:
- **`--base <path>`** (required): The base directory of the project where the package is located.
- **`--runtime <package>`**: Parent package for models to use in generated imports.
- **`--model-library <typed-dict|pydantic>`**: Which model library to use (default: `typed-dict`).
- **`--extra-imports <pkg:Class>`**: Add extra imports to the generated client.
- **`--dynamo-access-module <module>`**: Custom module for DynamoDB access.

### Examples:

**Basic TypedDict generation:**
```bash
prismarine generate-client --base . myapp.db
```

**Pydantic generation:**
```bash
prismarine generate-client --base . myapp.db --model-library pydantic
```

## version Command

Prints the current version of Prismarine.

```bash
prismarine version
```
