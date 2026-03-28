# Design Spec: EasySAM Companion Agent Skill

**Date:** 2026-03-28
**Topic:** EasySAM Companion Agent Skill
**Status:** Approved

## 1. Overview
The `easysam-skill` is a companion agent skill designed to help AI coding agents build, manage, and deploy serverless applications using the EasySAM YAML-to-SAM generator. It enforces opinionated best practices, modular architecture, and rigorous validation.

## 2. Architecture & Organization
The skill enforces a modular "Module Pattern" for AWS resources:
- **Root `resources.yaml`**: Global settings (`prefix`, `tags`, `envvars`, `python`) and the `import` list.
- **Feature Modules**: Independent directories (e.g., `orders/`, `users/`) containing:
    - `easysam.yaml`: Local resource definitions.
    - `backend/`: Python handler code.
    - `common/`: Shared logic.
- **Modularity Rule**: Agents must use the `import` mechanism to keep the project modular and maintainable.

## 3. Resource Implementation Workflow
A strict "Implement-Validate-Test" cycle for all resources:
1.  **Drafting**: Add resource to `easysam.yaml` using `!Ref` and `!GetAtt`.
2.  **Schema Validation**: Run `uv run easysam --environment dev inspect schema .` immediately.
3.  **Handler Scaffolding**: Create Python handlers in `backend/`. Move complex logic to `common/`.
4.  **Unit Testing**: Add `pytest` cases in `tests/` using standard AWS mocking patterns.
5.  **Cloud Check**: Run `uv run easysam inspect cloud .` to verify external ARNs and roles.

## 4. Deployment & Environment Strategy
- **Environment-First**: Always specify `--environment` (defaulting to `dev`).
- **Profile Safety**: Always use `--aws-profile` to avoid credential ambiguity.
- **Environment Overrides**: Use `deploy-context.yaml` for per-environment configuration (VPCs, ARNs).
- **Safe Updates**: Sequence: `inspect cloud` -> `generate` (preview `template.yml`) -> `deploy`.

## 5. CI/CD Implementation
- **Toolchain**: Optimized for GitHub Actions with `uv` and AWS SAM.
- **Security**: Prefer OIDC-based authentication over static IAM keys.
- **Gates**: `inspect schema` and `pytest` must pass before any deployment.
- **Mapping**: `main` branch deploys to `prod`; `develop`/PRs deploy to `dev`.
- **Artifacts**: Always run `easysam generate` in CI to produce the final `template.yml`.

## 6. Self-Review Notes
- **Placeholder scan**: No "TBD" or "TODO" items remain.
- **Consistency**: Architecture and workflows are aligned with the modular nature of EasySAM.
- **Scope**: Focused on the core developer lifecycle (Scaffold -> Add Resource -> Deploy -> CI/CD).
- **Ambiguity**: Workflows are explicitly sequenced to prevent common deployment errors.
