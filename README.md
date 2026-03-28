# EasySAM Skills

Companion agent skills for [EasySAM](https://github.com/scartill/easysam), an opinionated YAML-to-SAM generator for modular AWS serverless applications.

## Quick Start

Install the EasySAM skill directly from this repository:

```bash
npx skills add https://github.com/scartill/easysam-skills
```

After installation, reload your Gemini CLI session:
```bash
/skills reload
```

## Features

This skill helps AI coding agents to:
- **Scaffold**: Initialize new projects using EasySAM's modular pattern.
- **Resource Addition**: Add functions, tables, and queues with built-in schema validation.
- **Deployment**: Safely generate and deploy SAM templates with environment-specific overrides.
- **CI/CD**: Generate optimized GitHub Actions workflows using `uv` and OIDC.

## Documentation

- **[SKILL.md](easysam-skill/SKILL.md)**: Core workflows and instructions.
