---
name: easysam-skill
description: Build and deploy serverless applications using EasySAM. Use when the user wants to scaffold a new project, add AWS resources (Lambda, DynamoDB, S3, etc.), or set up CI/CD for an EasySAM project.
---

# EasySAM Skill

This skill helps you build and deploy serverless applications using the EasySAM YAML-to-SAM generator.

## Core Workflows

### 1. Scaffolding a Project
1. Run `uv run easysam init`.
2. Organize your project into modules (e.g., `orders/`, `users/`).
3. Create a `common/` directory in each module for shared logic.
4. Ensure the root `resources.yaml` imports your modules.
5. Clean up any boilerplate example files or placeholders before production.

### 2. Adding a Resource
1. Identify the target module and its `easysam.yaml`.
2. Add the resource definition (e.g., `tables`, `functions`).
3. Run schema validation: `uv run easysam --environment dev inspect schema .`.
4. Create handler code in `backend/`.
5. Add a unit test in `tests/`.

### 3. Deployment
1. Run `uv run easysam --environment dev --aws-profile <profile> inspect cloud .` to verify.
2. Generate the SAM template: `uv run easysam --environment dev generate .`.
3. Deploy: `uv run easysam --environment dev --aws-profile <profile> deploy .`.

## Reference Material
- **Patterns**: See [references/patterns.md](references/patterns.md) for common AWS recipes.
- **Troubleshooting**: See [references/troubleshooting.md](references/troubleshooting.md) for fixing schema and cloud errors.
- **CI/CD**: Use [assets/publish.yml](assets/publish.yml) for GitHub Actions.
