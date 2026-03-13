---
description: "Use when writing, reviewing, or modifying AWS CloudFormation templates. Covers template structure, naming conventions, required tags, security guardrails, and deployment safety rules."
applyTo: "**/*.yaml, **/*.json"
---

# CloudFormation Template Standards

## Template Structure

Organize sections in this order:

1. `AWSTemplateFormatVersion`
2. `Description` — one-line summary of what the stack provisions
3. `Metadata` (if needed)
4. `Parameters` — inputs with `Type`, `Description`, `Default`, and `AllowedValues` where applicable
5. `Mappings` (if needed)
6. `Conditions` (if needed)
7. `Resources`
8. `Outputs` — export key endpoints, ARNs, and resource IDs

Never omit `Description` or `Outputs`.

## Naming Conventions

- **Stack names**: `<app>-<env>-<service>` (e.g., `myapp-prod-api`)
- **Logical resource IDs**: PascalCase, descriptive (e.g., `ApiTaskDefinition`, `WebBucketPolicy`)
- **Parameter names**: PascalCase matching the resource they configure (e.g., `DesiredTaskCount`, `LambdaMemorySize`)
- **Export names**: `<stack-name>-<resource>` using `!Sub` (e.g., `!Sub "${AWS::StackName}-VpcId"`)

## Parameters

- Always include `Environment` parameter with `AllowedValues: [dev, staging, prod]`
- Use `AWS::SSM::Parameter::Value<String>` for secrets and environment-specific config — never hardcode sensitive values
- Add `ConstraintDescription` for parameters with validation rules
- Group related parameters using `AWS::CloudFormation::Interface` metadata

## Required Tags

Every taggable resource must include these tags:

```yaml
Tags:
  - Key: Environment
    Value: !Ref Environment
  - Key: Application
    Value: !Ref AWS::StackName
  - Key: ManagedBy
    Value: CloudFormation
```

Add `Team` and `CostCenter` tags when the values are known.

## Security Guardrails

- **No public S3 buckets**: Always set `PublicAccessBlockConfiguration` with all four blocks enabled
- **Encryption at rest**: Enable `SSESpecification` on DynamoDB, `BucketEncryption` on S3, `StorageEncrypted: true` on RDS
- **Least-privilege IAM**: Never use `Action: "*"` or `Resource: "*"` — scope to specific actions and ARNs
- **No hardcoded secrets**: Use `AWS::SSM::Parameter::Value`, `AWS::SecretsManager::Secret`, or `!Ref` to parameters — never inline credentials, API keys, or passwords
- **Security groups**: Never allow `0.0.0.0/0` on non-HTTP/HTTPS ports. Restrict SSH (22) to specific CIDRs or use SSM Session Manager instead
- **HTTPS only**: Configure ALB listeners to redirect HTTP → HTTPS. Use `ViewerProtocolPolicy: redirect-to-https` on CloudFront

## Deployment Safety Rules

- **Always review the changeset** before executing a deployment — never use `--no-execute-changeset` in production without human approval
- **Never deploy directly to production** — deploy to dev or staging first and verify
- **Use `DeletionPolicy: Retain`** on stateful resources (databases, S3 buckets with data, EFS)
- **Use `UpdateReplacePolicy: Retain`** alongside `DeletionPolicy` for resources that could be replaced during updates
- **Enable termination protection** on production stacks
- **Set stack rollback on failure**: Do not disable automatic rollback (`--disable-rollback`) in production

## Resource Ordering

- Use `DependsOn` only when CloudFormation cannot infer the dependency from `!Ref` or `!GetAtt`
- Prefer implicit dependencies (`!Ref`, `!GetAtt`) over explicit `DependsOn`

## Outputs

Always output:
- Primary endpoint URL or ARN of the deployed service
- Resource IDs needed by dependent stacks
- Use `Export` for cross-stack references
