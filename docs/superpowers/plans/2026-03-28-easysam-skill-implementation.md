# EasySAM Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a specialized agent skill for building and deploying serverless applications with EasySAM.

**Architecture:** A modular skill with a core `SKILL.md` for workflows, `references/` for patterns and troubleshooting, and `assets/` for CI/CD templates.

**Tech Stack:** Markdown, EasySAM CLI, GitHub Actions.

---

### Task 1: Core SKILL.md Implementation

**Files:**
- Modify: `easysam-skill/SKILL.md`

- [ ] **Step 1: Write the SKILL.md body**
Include workflows for Scaffolding, Resource Addition, and Deployment.

```markdown
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
```

- [ ] **Step 2: Commit**
```bash
git add easysam-skill/SKILL.md
git commit -m "feat(skill): implement core SKILL.md for easysam"
```

---

### Task 2: Resource Patterns Reference

**Files:**
- Create: `easysam-skill/references/patterns.md`

- [ ] **Step 1: Write the patterns reference**
Include recipes for common resource configurations.

```markdown
# EasySAM Resource Patterns

## Lambda + DynamoDB (with IAM)
```yaml
functions:
  my-function:
    uri: backend/handler.py
    tables:
      - !Ref MyTable
    envvars:
      TABLE_NAME: !Ref MyTable

tables:
  MyTable:
    attributes:
      - name: id
        hash: true
```

## SQS-Triggered Lambda
```yaml
queues:
  task-queue:

functions:
  worker:
    uri: backend/worker.py
    polls:
      - name: !Ref task-queue
        batchsize: 10
```

## Scheduled Cleanup
```yaml
functions:
  cleanup:
    uri: backend/cleanup.py
    schedule: "rate(1 day)"
```
```

- [ ] **Step 2: Commit**
```bash
git add easysam-skill/references/patterns.md
git commit -m "docs(skill): add resource patterns for easysam"
```

---

### Task 3: Troubleshooting Guide

**Files:**
- Create: `easysam-skill/references/troubleshooting.md`

- [ ] **Step 1: Write the troubleshooting guide**
Include common error patterns and fixes.

```markdown
# EasySAM Troubleshooting

## Schema Errors (`inspect schema`)
- **"is a required property"**: Check the `resources.yaml` or `easysam.yaml` for missing keys (e.g., `prefix`, `public` for buckets).
- **"is not allowed"**: You added a key not supported by the schema. Check `RESOURCE_REFERENCE.md`.

## Cloud Errors (`inspect cloud`)
- **"Resource not found"**: The ARN or Name provided in `deploy-context.yaml` does not exist in the target AWS account/region.
- **"Access Denied"**: Your AWS profile lacks permissions to describe the resource.

## Generation Errors (`generate`)
- **Jinja2 Template Error**: Check for typos in `!Ref` or `!GetAtt` names.
```

- [ ] **Step 2: Commit**
```bash
git add easysam-skill/references/troubleshooting.md
git commit -m "docs(skill): add troubleshooting guide for easysam"
```

---

### Task 4: CI/CD Asset

**Files:**
- Create: `easysam-skill/assets/publish.yml`

- [ ] **Step 1: Create the GitHub Actions workflow asset**

```yaml
name: Deploy EasySAM Application

on:
  push:
    branches: [ main, develop ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Install uv
        run: curl -LsSf https://astral.sh/uv/install.sh | sh
      - name: Setup Python
        run: uv python install
      - name: Install dependencies
        run: uv sync
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ secrets.AWS_REGION }}
      - name: Validate Schema
        run: uv run easysam --environment dev inspect schema .
      - name: Run Tests
        run: uv run pytest
      - name: Generate and Deploy
        run: |
          uv run easysam --environment dev generate .
          uv run easysam --environment dev deploy .
```

- [ ] **Step 2: Commit**
```bash
git add easysam-skill/assets/publish.yml
git commit -m "feat(skill): add GitHub Actions publish asset"
```

---

### Task 5: Packaging & Installation Guide

**Files:**
- Modify: `easysam-skill/SKILL.md` (Self-Correction: Add cleanup of example files)

- [ ] **Step 1: Clean up example files**
```bash
rm easysam-skill/scripts/example_script.cjs
rm easysam-skill/references/example_reference.md
rm easysam-skill/assets/example_asset.txt
```

- [ ] **Step 2: Package the skill**
Run the packaging script to create `easysam-skill.skill`.

```bash
node C:\Users\boris\AppData\Roaming\npm\node_modules\@google\gemini-cli\node_modules\@google\gemini-cli-core\dist\src\skills\builtin\skill-creator\scripts\package_skill.cjs easysam-skill
```

- [ ] **Step 3: Commit final state**
```bash
git add .
git commit -m "feat(skill): package easysam-skill"
```
