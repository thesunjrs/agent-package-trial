# Elastic Beanstalk Deployment

## Prerequisites

- EB CLI installed: `eb --version` (optional but recommended)
- Application and environment already created (via CloudFormation or EB CLI)

## CloudFormation Resources

- `AWS::ElasticBeanstalk::Application`
- `AWS::ElasticBeanstalk::ApplicationVersion`
- `AWS::ElasticBeanstalk::Environment` — solution stack, option settings
- `AWS::ElasticBeanstalk::ConfigurationTemplate` (optional)

## Package

1. Create a source bundle (ZIP) of the application:
   ```
   zip -r app-<version>.zip . -x "*.git*" "node_modules/*" ".env"
   ```
2. Upload to S3:
   ```
   aws s3 cp app-<version>.zip s3://<bucket>/eb-deployments/app-<version>.zip
   ```

## Deploy

1. Create a new application version:
   ```
   aws elasticbeanstalk create-application-version \
     --application-name <app-name> \
     --version-label <version> \
     --source-bundle S3Bucket=<bucket>,S3Key=eb-deployments/app-<version>.zip
   ```
2. Update the environment to use it:
   ```
   aws elasticbeanstalk update-environment \
     --environment-name <env-name> \
     --version-label <version>
   ```
3. Wait for environment to be ready:
   ```
   aws elasticbeanstalk wait environment-updated --environment-name <env-name>
   ```

Or with EB CLI:
```
eb deploy <env-name>
```

## Verify

1. Check environment health:
   ```
   aws elasticbeanstalk describe-environment-health \
     --environment-name <env-name> \
     --attribute-names All
   ```
   Expected: `"HealthStatus": "Ok"`, `"Color": "Green"`
2. Get the environment URL:
   ```
   aws elasticbeanstalk describe-environments --environment-names <env-name> \
     --query "Environments[0].CNAME"
   ```
3. Curl the health endpoint.

## Rollback

Deploy the previous version label:
```
aws elasticbeanstalk update-environment \
  --environment-name <env-name> \
  --version-label <previous-version>
```
