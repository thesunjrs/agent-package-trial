---
description: "Deploy an application to AWS using the CLI. Runs the full workflow: build, validate CloudFormation, deploy stack, verify, and rollback on failure."
agent: "agent"
argument-hint: "Describe what to deploy and which AWS service to target (e.g., 'deploy the API to ECS Fargate in us-east-1')"
tools: ["terminal", "search", "readFile"]
---

Deploy the application to AWS using the CLI.

## Instructions

Follow the [aws-deployment skill](./../skills/aws-deployment/SKILL.md) end-to-end:

1. **Prerequisites** — verify AWS CLI, credentials (`aws sts get-caller-identity`), region, and any service-specific tools (Docker, kubectl).
2. **Build & Package** — run the project's build step and tests. Abort if tests fail.
3. **Validate CloudFormation** — lint and create a changeset. Show the changeset summary and **ask for confirmation** before executing.
4. **Deploy** — execute the deployment and monitor stack events for failures.
5. **Verify** — check stack status, run the service-specific health check, and report endpoints/outputs.
6. **Rollback** — if verification fails, roll back and report the failure cause.

Follow [CloudFormation template standards](../instructions/cloudformation.instructions.md) when creating or modifying templates.

**Always ask for confirmation before executing a changeset or any destructive action.**
