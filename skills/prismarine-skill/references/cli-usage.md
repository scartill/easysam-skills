# Prismarine CLI Usage

The Prismarine CLI is used to generate the client code from your models.

## generate-client Command

Generates a `prismarine_client.py` file.

### Arguments:
- **`CLUSTER_PACKAGE`**: The Python package containing your models (e.g., `common`). The CLI expects to import `common.models` and find a `Cluster` instance there.

### Options:
- **`--base <path>`** (required): The base directory of the project where the package is located.
- **`--model-library <typed-dict|pydantic>`**: Which model library to use (default: `typed-dict`).
- **`--runtime <package>`**: Parent package for models to use in generated imports.
- **`--extra-imports <pkg:Class>`**: Add extra imports to the generated client.
- **`--dynamo-access-module <module>`**: Custom module for DynamoDB access.

### Recommended Usage:

**Pydantic generation (common for EasySAM projects):**
```bash
uv run prismarine generate-client common --base . --model-library pydantic
```

## version Command

Prints the current version of Prismarine.

```bash
uv run prismarine version
```
