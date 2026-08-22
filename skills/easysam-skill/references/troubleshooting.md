# EasySAM Troubleshooting Guide

This guide details common validation, cloud, dependency, and generation issues encountered when working with EasySAM.

## 1. Schema Validation Errors (`inspect schema`)
- **"is a required property"**:
  - *Cause*: A mandatory top-level key is missing from `resources.yaml` or `easysam.yaml`.
  - *Fix*: Check required fields (`prefix` in `resources.yaml`, `name` in `lambda:`, `public` in bucket shortcuts).
- **"is not allowed" / "Additional properties are not allowed"**:
  - *Cause*: An invalid key or syntax structure was used.
  - *Fixes*:
    - Ensure `envvars` is placed inside `lambda: -> resources: -> envvars:` (NOT directly under `lambda:`).
    - Ensure `integration:` (not `api:`) is used for HTTP Lambdas.
    - Do NOT use `!Ref` or `!GetAtt` for table/bucket references; use bare string names.

## 2. Cloud Inspection Errors (`inspect cloud`)
- **"Resource not found"**:
  - *Cause*: The specified ARN or name in `deploy-context.yaml` does not exist in the target AWS account/region.
  - *Fix*: Verify the AWS region, account ID, and profile provided via `--aws-profile`.
- **"Access Denied" / "Expired Token"**:
  - *Cause*: Invalid or expired AWS credentials.
  - *Fix*: Refresh AWS SSO/IAM credentials and pass `--aws-profile <profile_name>`.

## 3. Template Generation Errors (`generate`)
- **Jinja2 Template Render Error**:
  - *Cause*: Syntax error in variable interpolation or SSM parameter string.
  - *Fix*: Check SSM parameter format: `{{resolve:ssm:/path/to/param}}`. Do NOT use `!Param`.
- **Missing Module Import Error**:
  - *Cause*: Root `resources.yaml` does not list subdirectories containing `easysam.yaml`.
  - *Fix*: Add the relative path to `import:` array in `resources.yaml` (e.g., `import: [backend]`).

## 4. Lambda Packaging & Dependency Errors
- **"ModuleNotFoundError" in Lambda runtime**:
  - *Cause*: Dependencies listed in `pyproject.toml` instead of Lambda packaging directory.
  - *Fix*: Place all third-party Lambda runtime dependencies in `sam/thirdparty/requirements.txt`.
- **Symlink / Common Import Errors in Git**:
  - *Cause*: Symlinked `common/` folder tracked by Git or missing from module build.
  - *Fix*: Add `**/common/` to `.gitignore` inside Lambda directories.
