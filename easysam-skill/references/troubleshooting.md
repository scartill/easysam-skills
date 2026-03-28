# EasySAM Troubleshooting

## Schema Errors (`inspect schema`)
- **"is a required property"**: Check the `resources.yaml` or `easysam.yaml` for missing keys (e.g., `prefix`, `public` for buckets).
- **"is not allowed"**: You added a key not supported by the schema. Check `RESOURCE_REFERENCE.md`.

## Cloud Errors (`inspect cloud`)
- **"Resource not found"**: The ARN or Name provided in `deploy-context.yaml` does not exist in the target AWS account/region.
- **"Access Denied"**: Your AWS profile lacks permissions to describe the resource.

## Generation Errors (`generate`)
- **Jinja2 Template Error**: Check for typos in `!Ref` or `!GetAtt` names.
