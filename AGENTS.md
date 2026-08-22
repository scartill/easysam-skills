# EasySAM Skills - Project Context

This project is a repository for creating and managing specialized agent skills for **EasySAM** and **Prismarine**, providing opinionated workflows for modular AWS serverless applications and DynamoDB modeling.

## Project Overview

The project provides companion agent skills designed to help AI coding agents build, manage, and deploy serverless applications and database models.

### Key Components
- **`skills/easysam-skill/`**: The core skill for EasySAM infrastructure management.
    - `SKILL.md`: Core workflows for scaffolding, resource addition, and deployment.
    - `references/`: Patterns recipes and troubleshooting guides.
    - `assets/`: Pre-configured CI/CD workflow templates.
- **`skills/prismarine-skill/`**: The development guide for the Prismarine DynamoDB ORM.
    - `SKILL.md`: Workflows for model definition, client generation, and CRUD operations.
- **`docs/superpowers/`**: Contains the design specs (`specs/`) and implementation plans (`plans/`) used to build the skills.

## Development Workflows

### 1. Modifying Skills
When updating skills, follow the design principles outlined in `docs/superpowers/specs/2026-03-28-easysam-skill-design.md`. 
- For `easysam-skill`, ensure any new resource patterns are added to `references/patterns.md`.
- Keep `SKILL.md` files concise and focused on high-level procedural guardrails.

### 2. Packaging Skills
To package a skill into a `.skill` file for distribution:
```powershell
# Run the packaging script (adjust path to gemini-cli-core as needed)
node <path-to-gemini-cli-core>\dist\src\skills\builtin\skill-creator\scripts\package_skill.cjs <skill-directory-name>
```

### 3. Testing Skills
After packaging, install the skill locally to verify its behavior:
```powershell
gemini skills install <skill-file-name>.skill --scope workspace
# Then run /skills reload in the interactive session
```

## Development Conventions

- **Modular Architecture**: Always promote the "Module Pattern" (separate directories with `easysam.yaml` files) over monolithic configurations.
- **Validation Gates**: The `easysam-skill` must always instruct agents to run `inspect schema` and `inspect cloud` before deployment.
- **CI/CD Best Practices**: Prefer OIDC-based authentication and `uv`-optimized pipelines in the generated assets.
- **Test-Driven Implementation**: Resource additions should always be accompanied by a corresponding unit test.
- **Type-Safe Modeling**: The `prismarine-skill` should always promote the use of the `prismarine generate-client` CLI for generating type-safe Python client code.
