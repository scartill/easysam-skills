# EasySAM Skills

Companion agent skills for [EasySAM](https://github.com/scartill/easysam) and [Prismarine](https://github.com/scartill/prismarine), providing opinionated workflows for building modular AWS serverless applications.

## Available Skills

### 1. [EasySAM Skill](skills/easysam-skill/SKILL.md)
The core skill for scaffolding, managing, and deploying modular AWS SAM applications.
- **Scaffold**: Initialize new projects using EasySAM's modular pattern.
- **Resource Addition**: Add functions, tables, and queues with built-in schema validation.
- **Deployment**: Safely generate and deploy SAM templates with environment-specific overrides.
- **CI/CD**: Generate optimized GitHub Actions workflows using `uv` and OIDC.

### 2. [Prismarine Skill](skills/prismarine-skill/SKILL.md)
A specialized development guide for the Prismarine DynamoDB ORM. Use this when defining models, generating type-safe client code, or performing database operations.
- **Model Definition**: Guidance for `@c.model`, `@c.index`, and schema design.
- **Client Generation**: Instructions for the `prismarine generate-client` CLI.
- **CRUD Operations**: High-level API patterns for type-safe database access.
- **EasySAM Integration**: Patterns for linking DynamoDB models to serverless resources.

## Quick Start

Install the skills directly from this repository:

```bash
npx skills add https://github.com/scartill/easysam-skills
```

After installation, reload your Gemini CLI session:
```bash
/skills reload
```

## Documentation

- **[EasySAM Workflows](skills/easysam-skill/SKILL.md)**
- **[Prismarine Workflows](skills/prismarine-skill/SKILL.md)**
