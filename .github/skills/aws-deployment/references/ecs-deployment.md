# ECS (Fargate/EC2) Deployment

## Build & Push Image

1. Authenticate Docker to ECR:
   ```
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
   ```
2. Build and tag:
   ```
   docker build -t <repo-name>:<tag> .
   docker tag <repo-name>:<tag> <account>.dkr.ecr.<region>.amazonaws.com/<repo-name>:<tag>
   ```
3. Push:
   ```
   docker push <account>.dkr.ecr.<region>.amazonaws.com/<repo-name>:<tag>
   ```

## CloudFormation Resources

Key resources in the template:

- `AWS::ECS::Cluster`
- `AWS::ECS::TaskDefinition` — container definitions, CPU/memory, IAM roles
- `AWS::ECS::Service` — desired count, load balancer config, deployment configuration
- `AWS::ElasticLoadBalancingV2::TargetGroup` (if using ALB)

## Deploy

After pushing the image, update the CloudFormation stack. If only the image tag changed and the template is unchanged, force a new deployment:

```
aws ecs update-service --cluster <cluster> --service <service> --force-new-deployment
```

## Verify

1. Wait for service stability:
   ```
   aws ecs wait services-stable --cluster <cluster> --services <service>
   ```
2. Check running task count:
   ```
   aws ecs describe-services --cluster <cluster> --services <service> \
     --query "services[0].{desired:desiredCount,running:runningCount}"
   ```
3. Hit the health endpoint through the load balancer URL.

## Rollback

1. Revert the task definition to the previous revision:
   ```
   aws ecs update-service --cluster <cluster> --service <service> \
     --task-definition <previous-task-def-arn>
   ```
2. Wait for stability again.
