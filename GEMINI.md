# EasySAM Skills - Project Context

This project is a repository for creating and managing specialized agent skills for **EasySAM**, an opinionated YAML-to-SAM generator for modular AWS serverless applications.

## Project Overview

The primary output of this project is the `easysam-skill`, a companion agent skill designed to help AI coding agents build, manage, and deploy serverless applications using EasySAM's modular architecture.

### Key Components
- **`easysam-skill/`**: The source directory for the agent skill.
    - `SKILL.md`: Core workflows for scaffolding, resource addition, and deployment.
    - `references/`: Patterns recipes and troubleshooting guides.
    - `assets/`: Pre-configured CI/CD workflow templates.
- **`docs/superpowers/`**: Contains the design specs (`specs/`) and implementation plans (`plans/`) used to build the skills.
- **`easysam-skill.skill`**: The packaged, distributable skill file.

## Development Workflows

### 1. Modifying the Skill
When updating the `easysam-skill`, follow the design principles outlined in `docs/superpowers/specs/2026-03-28-easysam-skill-design.md`. 
- Ensure any new resource patterns are added to `easysam-skill/references/patterns.md`.
- Keep `SKILL.md` concise and focused on high-level procedural guardrails.

### 2. Packaging the Skill
To package the skill into a `.skill` file for distribution:
```powershell
# Run the packaging script (adjust path to gemini-cli-core as needed)
node <path-to-gemini-cli-core>\dist\src\skills\builtin\skill-creator\scripts\package_skill.cjs easysam-skill
```

### 3. Testing the Skill
After packaging, install the skill locally to verify its behavior:
```powershell
gemini skills install easysam-skill.skill --scope workspace
# Then run /skills reload in the interactive session
```

## Development Conventions

- **Modular Architecture**: Always promote the "Module Pattern" (separate directories with `easysam.yaml` files) over monolithic configurations.
- **Validation Gates**: The skill must always instruct agents to run `inspect schema` and `inspect cloud` before deployment.
- **CI/CD Best Practices**: Prefer OIDC-based authentication and `uv`-optimized pipelines in the generated assets.
- **Test-Driven Implementation**: Resource additions should always be accompanied by a corresponding unit test.
