---
name: aws-deployment
description: "Deploy applications to AWS using CloudFormation and the AWS CLI. USE FOR: deploying to ECS Fargate/EC2, Lambda, EC2, EKS, S3+CloudFront, Elastic Beanstalk; creating or updating CloudFormation stacks; pre-deploy validation; post-deploy verification; rollback. DO NOT USE FOR: initial AWS account setup, IAM policy authoring, or cost optimization."
argument-hint: "Describe what to deploy and which AWS service to target (e.g., 'deploy the API to ECS Fargate in us-east-1')"
---

# AWS Deployment

Deploy applications to AWS services using CloudFormation and the AWS CLI from a local machine.

## When to Use

- Deploy a new service or update an existing one on AWS
- Create, update, or delete CloudFormation stacks
- Verify a deployment succeeded and roll back if it didn't
- Package and push artifacts (Docker images, Lambda ZIPs, static assets)

## Prerequisites Check

Before starting any deployment, verify:

1. **AWS CLI installed**: `aws --version` (v2 required)
2. **Credentials configured**: `aws sts get-caller-identity` must return the correct account/role
3. **Target region set**: `aws configure get region` or pass `--region` explicitly
4. **Docker** (ECS/EKS only): `docker --version`
5. **kubectl** (EKS only): `kubectl version --client`

If any prerequisite fails, stop and resolve it before proceeding.

## Decision: Which Service?

| Signal | Target Service | Reference |
|--------|---------------|-----------|
| Containerised app, long-running | **ECS (Fargate/EC2)** | [ECS guide](./references/ecs-deployment.md) |
| Event-driven, short-lived function | **Lambda** | [Lambda guide](./references/lambda-deployment.md) |
| Full OS control, SSH/SSM access | **EC2 (direct)** | [EC2 guide](./references/ec2-deployment.md) |
| Kubernetes workloads | **EKS** | [EKS guide](./references/eks-deployment.md) |
| Static website or SPA | **S3 + CloudFront** | [S3+CF guide](./references/s3-cloudfront-deployment.md) |
| Quick PaaS-style deployment | **Elastic Beanstalk** | [EB guide](./references/elastic-beanstalk-deployment.md) |

If the user hasn't specified a target, ask which service they're deploying to.

## Universal Deployment Procedure

Regardless of target service, every deployment follows these phases:

### Phase 1 — Build & Package

1. Run the project's build step (e.g., `npm run build`, `dotnet publish`, `mvn package`).
2. Run tests: `npm test` / `dotnet test` / `pytest` — abort if tests fail.
3. Package the artifact for the target (Docker image, ZIP, static files).

### Phase 2 — Validate CloudFormation

1. Lint the template:
   ```
   aws cloudformation validate-template --template-body file://template.yaml
   ```
2. Review the changeset before applying:
   ```
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name <stack-name> \
     --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
     --no-execute-changeset
   ```
3. Show the user the changeset summary and wait for confirmation before executing.

### Phase 3 — Deploy

1. Execute the CloudFormation deployment (or the service-specific deployment command from the reference guide).
2. Monitor stack events:
   ```
   aws cloudformation describe-stack-events --stack-name <stack-name> \
     --query "StackEvents[?ResourceStatus=='CREATE_FAILED' || ResourceStatus=='UPDATE_FAILED']"
   ```
3. Wait for completion:
   ```
   aws cloudformation wait stack-create-complete --stack-name <stack-name>
   ```
   (Use `stack-update-complete` for updates.)

### Phase 4 — Verify

1. Check stack status:
   ```
   aws cloudformation describe-stacks --stack-name <stack-name> \
     --query "Stacks[0].StackStatus"
   ```
   Expected: `CREATE_COMPLETE` or `UPDATE_COMPLETE`.
2. Run the service-specific health check (see reference guide for the target service).
3. Report the deployment outcome: stack outputs, endpoint URLs, resource IDs.

### Phase 5 — Rollback (if verification fails)

1. For CloudFormation stacks that are in a `*_FAILED` state:
   ```
   aws cloudformation rollback-stack --stack-name <stack-name>
   ```
2. For service-level rollback (e.g., ECS service revert, Lambda alias switch), follow the service-specific reference guide.
3. Investigate the failure using CloudFormation events and CloudWatch logs before retrying.

## Environment Strategy

Use a naming convention to separate environments:

| Environment | Stack name pattern | Account/Region |
|-------------|-------------------|----------------|
| Dev | `<app>-dev-<service>` | Same account, same region |
| Staging | `<app>-staging-<service>` | Same or separate account |
| Production | `<app>-prod-<service>` | Separate account recommended |

Always deploy to a non-production environment first and verify before promoting.

## Common Failure Patterns

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| `InsufficientCapabilities` | Missing `--capabilities` flag | Add `CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` |
| `ResourceNotReady` | Dependency not yet available | Add `DependsOn` or check ordering |
| `UPDATE_ROLLBACK_COMPLETE` | Previous update failed | Fix the template and redeploy |
| `Rate exceeded` | Too many API calls | Add retries with backoff |
| `AccessDenied` | IAM permissions missing | Check the deploying role's policies |
