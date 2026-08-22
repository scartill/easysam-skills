# Prismarine CLI Usage Reference

While EasySAM automatically runs Prismarine client generation during `easysam generate` and `easysam deploy`, you can also invoke the `prismarine` CLI directly for local development and testing.

---

## `generate-client` Command

Generates `prismarine_client.py` from model definitions in a package.

### Command Syntax:

```bash
uv run prismarine generate-client <PACKAGE> --base <BASE_DIR> [OPTIONS]
```

### Arguments:
- **`PACKAGE`**: Subpackage name under base containing `models.py` (e.g. `myobject`). The CLI imports `<BASE_DIR>.<PACKAGE>.models`.

### Key Options:

| Option | Description | Example |
| --- | --- | --- |
| `--base <path>` | Base root directory containing the package (**required**). | `--base common` |
| `--model-library <typed-dict\|pydantic>` | Model representation for generated client. | `--model-library pydantic` |
| `--dynamo-access-module <module>` | Custom access module import path. | `--dynamo-access-module common.dynamo_access` |
| `--extra-imports <module:Class>` | Additional imports to include in client. | `--extra-imports common.models:CustomType` |

---

## Examples

### TypedDict Generation:
```bash
uv run prismarine generate-client myobject --base common --model-library typed-dict
```

### Pydantic Generation:
```bash
uv run prismarine generate-client myobject --base common --model-library pydantic --dynamo-access-module common.dynamo_access
```

---

## `version` Command

Prints the installed version of Prismarine:

```bash
uv run prismarine version
```
