# EasySAM Troubleshooting Guide

This guide details common validation, cloud, deployment, routing, and packaging issues encountered when working with EasySAM.

## 1. Cardinal Directives & Deployment Failures

### Never Edit `template.yml` Directly
- **Issue**: Manual edits to `template.yml` or `template.yaml` disappear after running `easysam generate` or `easysam deploy`.
- **Cause**: `template.yml` is an ephemeral artifact produced dynamically from your EasySAM YAML files (`resources.yaml`, `easysam.yaml`, `deploy-context.yaml`).
- **Fix**: Make all resource, environment variable, or IAM role modifications inside `resources.yaml` or module-level `easysam.yaml`.

### Stack Stuck or Repeated Deployment Failures (Circuit Breaker)
- **Issue**: `easysam deploy` or `sam deploy` hangs, fails repeatedly, or gets stuck in `UPDATE_ROLLBACK_IN_PROGRESS` / `UPDATE_IN_PROGRESS`.
- **Rule**: If a deployment fails or gets stuck **2 or more times**:
  - **STOP retrying commands immediately.** Do not run duplicated deploy commands or loop background scripts.
  - Fetch the exact CloudFormation error events using `aws cloudformation describe-stack-events --stack-name <stack_name>`.
  - Report the stuck stack state and exact error logs to the user, offering options to proceed (e.g. `easysam delete --force`, manual AWS Console rollback, or fixing resource name lock contention).

---

## 2. Routing & HTTP Integration Errors

### FastAPI Returning 404 for Sub-Routes
- **Issue**: API Gateway returns 404 for endpoint paths like `/api/v1/users/123`, but root `/api/v1` works.
- **Cause**: The HTTP Lambda's `integration:` block is non-greedy (`greedy: false` or omitted), so sub-paths are not forwarded to FastAPI's internal router.
- **Fix**: Set `greedy: true` under `integration:` in `easysam.yaml`:
  ```yaml
  lambda:
    name: api-handler
    integration:
      path: /api/v1
      greedy: true
      open: true
  ```

---

## 3. Schema Validation Errors (`inspect schema`)
- **"is a required property"**:
  - *Cause*: A mandatory top-level key is missing from `resources.yaml` or `easysam.yaml`.
  - *Fix*: Check required fields (`prefix` in `resources.yaml`, `name` in `lambda:`, `public` in bucket shortcuts).
- **"is not allowed" / "Additional properties are not allowed"**:
  - *Cause*: An invalid key or syntax structure was used.
  - *Fixes*:
    - Ensure `envvars` is placed inside `lambda: -> resources: -> envvars:` (NOT directly under `lambda:`).
    - Ensure `integration:` (not `api:`) is used for HTTP Lambdas.
    - Do NOT use `!Ref` or `!GetAtt` for table/bucket references; use bare string names.

---

## 4. Cloud Inspection Errors (`inspect cloud`)
- **"Resource not found"**:
  - *Cause*: The specified ARN or name in `deploy-context.yaml` does not exist in the target AWS account/region.
  - *Fix*: Verify the AWS region, account ID, and profile provided via `--aws-profile`.
- **"Access Denied" / "Expired Token"**:
  - *Cause*: Invalid or expired AWS credentials.
  - *Fix*: Refresh AWS SSO/IAM credentials and pass `--aws-profile <profile_name>`.

---

## 5. Lambda Packaging & Dependency Errors
- **"ModuleNotFoundError" in Lambda runtime**:
  - *Cause*: Dependencies listed in `pyproject.toml` instead of Lambda packaging directory.
  - *Fix*: Place all third-party Lambda runtime dependencies in `sam/thirdparty/requirements.txt`.
- **Symlink / Common Import / Prismarine Errors in Git**:
  - *Cause*: `common/` or `prismarine_clients/` tracked by Git or dirty across branches.
  - *Fix*: Ensure `.gitignore` contains `**/common/` and `**/prismarine_clients/`.
