---
name: easysam-skill
description: Build and deploy serverless applications using EasySAM. Use when the user wants to scaffold a new project, add AWS resources (Lambda, DynamoDB, S3, etc.), or set up CI/CD for an EasySAM project.
---

# EasySAM Skill

This skill helps you build and deploy serverless applications using the EasySAM YAML-to-SAM generator.

## Standard Project Hierarchy

EasySAM encourages a modular "Module Pattern" for organizing AWS resources. The baseline hierarchy follows this structure:

```text
my-project/
├── resources.yaml            # Global settings (prefix, tags) and module imports
├── sam/
│   └── thirdparty/
│       └── requirements.txt  # Runtime dependencies (FastAPI, Prismarine, etc.)
├── pyproject.toml            # Dev dependencies (pytest, ruff, easysam)
├── backend/                  # Main module (imported by resources.yaml)
│   ├── database/             # Data resources (DynamoDB, RDS)
│   │   └── easysam.yaml
│   └── function/             # Compute resources (Lambdas)
│       └── my-function/
│           ├── easysam.yaml  # Local resource definition
│           └── index.py      # Lambda handler code
├── common/                   # Shared logic (referenced by multiple modules)
│   └── utils.py
└── tests/                    # Project-level tests (pytest)
    └── test_myapp.py
```

### Dependency Management
- **Runtime Dependencies**: Place in `sam/thirdparty/requirements.txt`. These are packaged into the Lambda deployment artifact.
- **Dev Dependencies**: Use `pyproject.toml` with `[dependency-groups] dev`.
- **Project Dependencies**: Keep `pyproject.toml [project].dependencies` empty (`dependencies = []`).

### Git Configuration
To prevent local symlinks or copied code from being tracked, add `**/common/` to the `.gitignore` file within the directory containing your Lambda code (usually `backend/`).

## Core Workflows

### 1. Scaffolding a Project
1. Run `uv run easysam init`.
2. Organize resources by type under `backend/` or by feature (e.g., `orders/`, `users/`).
3. Place shared helper functions in a root `common/` directory.
4. Ensure the root `resources.yaml` imports your top-level modules (e.g., `import: [backend]`).
5. Keep Lambda handler logic minimal; delegate complexity to `common/`.

### 2. Adding a Resource
1. Identify the target module (e.g., `backend/database/` or `backend/function/myfunc/`).
2. Add the resource definition to the local `easysam.yaml`.
3. Run schema validation: `uv run easysam --environment dev inspect schema .`.
4. Create handler code (if needed) in the same directory as the local `easysam.yaml`.
5. Add a unit test in `tests/`.

### 3. Deployment
1. Run `uv run easysam --environment dev --aws-profile <profile> inspect cloud .` to verify.
2. Generate the SAM template: `uv run easysam --environment dev generate .`.
3. Deploy: `uv run easysam --environment dev --aws-profile <profile> deploy .`.

## EasySAM YAML Syntax Rules

### Resource References
Use **bare names** for table and bucket references. **Do NOT use `!Ref`**.
```yaml
resources:
  tables:
    - MyTable      # Correct
  buckets:
    - my-bucket    # Correct
```

### Environment Variables
`envvars` must be a child of `resources:`, NOT a sibling of it under `lambda:`.
- **Bucket/Table Refs**: Use bare names.
- **SSM Parameters**: Use `{{resolve:ssm:/path/to/param}}`. **Do NOT use `!Param`**.
```yaml
lambda:
  resources:
    envvars:
      TABLE_NAME: MyTable
      BUCKET_NAME: my-bucket
      API_KEY: "{{resolve:ssm:/myapp/api-key}}"
```

### HTTP Integration
Use `integration:` (not `api:`). Each Lambda needs a unique path prefix.
```yaml
lambda:
  integration:
    path: /my-service
    open: true
```

## Reference Material
- **Patterns**: See [references/patterns.md](references/patterns.md) for common AWS recipes.
- **Troubleshooting**: See [references/troubleshooting.md](references/troubleshooting.md) for fixing schema and cloud errors.
- **CI/CD**: Use [assets/publish.yml](assets/publish.yml) for GitHub Actions.
