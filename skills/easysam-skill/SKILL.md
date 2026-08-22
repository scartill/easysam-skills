---
name: easysam-skill
description: Build and deploy modular serverless applications using the EasySAM YAML-to-SAM generator. Always use this skill whenever the user asks to scaffold a serverless project, configure AWS resources (Lambda, DynamoDB, S3, SQS, SNS, EventBridge poller), define resources.yaml or easysam.yaml, inspect schema or cloud settings, generate SAM templates, or set up GitHub Actions CI/CD pipelines for serverless applications, even if they don't explicitly mention 'EasySAM'.
---

# EasySAM Skill

This skill provides opinionated workflows and syntax rules for building, validating, and deploying serverless applications using the EasySAM YAML-to-SAM generator.

## Standard Project Hierarchy

EasySAM strictly enforces a modular "Module Pattern" for organizing AWS resources. Avoid monolithic configurations; divide applications into feature or resource modules:

```text
my-project/
├── resources.yaml            # Global configuration (prefix, tags, python, envvars) and module imports
├── deploy-context.yaml       # Environment overrides (dev, prod ARNs/VPCs)
├── sam/
│   └── thirdparty/
│       └── requirements.txt  # Runtime dependencies packaged into Lambda artifacts
├── pyproject.toml            # Dev dependencies (pytest, ruff, easysam)
├── backend/                  # Main module (imported by resources.yaml)
│   ├── database/             # Database resources module
│   │   └── easysam.yaml
│   └── function/             # Compute resources module
│       └── my-function/
│           ├── easysam.yaml  # Local resource definition
│           └── index.py      # Lambda handler code
├── common/                   # Shared application logic
│   └── utils.py
└── tests/                    # Unit & integration test suite (pytest)
    └── test_myapp.py
```

### Key Architectural Conventions
- **Modular Imports**: Root `resources.yaml` must list sub-modules under `import:` (e.g., `import: [backend]`).
- **Dependency Management**:
  - Place Lambda runtime packages in `sam/thirdparty/requirements.txt`.
  - Keep project dependencies empty in `pyproject.toml` (`[project] dependencies = []`) and place development tools under `[dependency-groups] dev`.
- **Git Configuration**: Place `**/common/` in the `.gitignore` of Lambda code directories to avoid tracking synced code.

## Core Developer Workflows

### 1. Scaffolding a New Application
1. Run `uv run easysam init` to initialize project baseline.
2. Structure modules by boundary (e.g., `backend/database/`, `backend/orders/`).
3. Define global settings in `resources.yaml` (set `prefix`, `tags`, `python`, `import`).
4. Place core business logic in `common/` and keep Lambda handlers minimal.

### 2. Adding a Resource (Implement-Validate-Test Cycle)
1. Add local resource definitions in the target module's `easysam.yaml`.
2. **Schema Validation Gate**: Immediately run `uv run easysam --environment dev inspect schema .` to validate YAML schema.
3. Write minimal Lambda handler code alongside the module's `easysam.yaml`.
4. Write accompanying unit tests in `tests/` using `pytest`.

### 3. Deployment Pipeline
1. **Cloud Verification Gate**: Run `uv run easysam --environment dev --aws-profile <profile> inspect cloud .` to verify external ARNs and roles.
2. **Template Preview**: Run `uv run easysam --environment dev generate .` to inspect the generated `template.yml`.
3. **Deploy**: Run `uv run easysam --environment dev --aws-profile <profile> deploy .`.

## EasySAM YAML Syntax Rules

### 1. Resource References
- **Tables & Buckets**: Refer to local DynamoDB tables and S3 buckets by bare name strings (`MyTable`, `my-bucket`). **Do NOT use `!Ref`**.

### 2. Environment Variables
- `envvars` MUST be defined under `resources:`, NOT as a sibling of `resources:` under `lambda:`.
- **SSM Parameters**: Use `{{resolve:ssm:/path/to/param}}`. **Do NOT use `!Param`**.

```yaml
lambda:
  name: my-service
  resources:
    tables:
      - MyTable
    buckets:
      - my-bucket
    envvars:
      TABLE_NAME: MyTable
      BUCKET_NAME: my-bucket
      API_KEY: "{{resolve:ssm:/myapp/api-key}}"
```

### 3. HTTP Integrations
- Use `integration:` (not `api:`). Ensure each HTTP Lambda has a unique path prefix.

```yaml
lambda:
  name: api-handler
  integration:
    path: /api/v1
    open: true
```

## Reference Material
- **Resource Recipes & Patterns**: See [references/patterns.md](references/patterns.md) for full YAML recipes (DynamoDB, S3, SQS, SNS, Poller, `resources.yaml`, `deploy-context.yaml`).
- **Troubleshooting**: See [references/troubleshooting.md](references/troubleshooting.md) for schema, cloud, and template resolution error fixes.
- **CI/CD Pipeline**: Use [assets/publish.yml](assets/publish.yml) for GitHub Actions OIDC deployment.
