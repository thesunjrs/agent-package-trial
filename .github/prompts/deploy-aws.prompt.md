---
description: "Choose deployment method for AWS. Presents options for CLI-based, GitHub Actions, or Jenkins deployments and delegates to the appropriate specialist agent."
agent: "agent"
argument-hint: "Describe what you want to deploy (e.g., 'static site to S3', 'API to ECS')"
---

Choose your AWS deployment method and I'll route you to the appropriate specialist.

## Available Deployment Methods

1. **AWS CLI (Local)** — Deploy directly from your machine using AWS CLI
   - Use when: Testing locally, quick deployments, manual operations
   - Agent: `@aws-cli-deployer`

2. **GitHub Actions** — Automated CI/CD deployments via GitHub workflows
   - Use when: Continuous deployment, team collaboration, automated testing before deploy
   - Agent: `@aws-github-actions-deployer`

3. **Jenkins** — Automated CI/CD deployments via Jenkins pipelines
   - Use when: Existing Jenkins infrastructure, enterprise CI/CD, complex approval workflows
   - Agent: `@aws-jenkins-deployer`

## Instructions

Ask the user which deployment method they prefer, then invoke the appropriate agent:

- If **CLI/local**: Invoke `@aws-cli-deployer` with the full deployment request
- If **GitHub Actions**: Invoke `@aws-github-actions-deployer` with the deployment request
- If **Jenkins**: Invoke `@aws-jenkins-deployer` with the deployment request

If the user's request mentions a CI/CD tool explicitly (e.g., "using GitHub Actions"), skip the question and route directly.

**Ask now**: "Which deployment method would you like to use: AWS CLI (local), GitHub Actions, or Jenkins?"
