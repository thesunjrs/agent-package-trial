# EC2 (Direct) Deployment

## Prerequisites

- SSM Agent installed on the target instance, or SSH key pair configured
- Security group allows inbound traffic on the application port

## CloudFormation Resources

- `AWS::EC2::Instance` or `AWS::AutoScaling::LaunchConfiguration`
- `AWS::EC2::SecurityGroup`
- `AWS::ElasticLoadBalancingV2::TargetGroup` (if behind ALB)

## Deploy via SSM (preferred over SSH)

1. Upload the artifact to S3:
   ```
   aws s3 cp ./build/app.tar.gz s3://<bucket>/deployments/app-<version>.tar.gz
   ```
2. Run a deployment command on the instance:
   ```
   aws ssm send-command \
     --instance-ids <instance-id> \
     --document-name "AWS-RunShellScript" \
     --parameters 'commands=["cd /opt/app","aws s3 cp s3://<bucket>/deployments/app-<version>.tar.gz .","tar xzf app-<version>.tar.gz","sudo systemctl restart app"]'
   ```
3. Check command status:
   ```
   aws ssm get-command-invocation --command-id <id> --instance-id <instance-id>
   ```

## Verify

1. Check the instance is healthy:
   ```
   aws ec2 describe-instance-status --instance-ids <instance-id> \
     --query "InstanceStatuses[0].InstanceState.Name"
   ```
2. Check application health via the endpoint or SSM:
   ```
   aws ssm send-command \
     --instance-ids <instance-id> \
     --document-name "AWS-RunShellScript" \
     --parameters 'commands=["curl -sf http://localhost:<port>/health"]'
   ```

## Rollback

Re-deploy the previous artifact version from S3 using the same SSM command pattern.
