# Lambda Deployment

## Package

1. Install production dependencies and create a ZIP:
   ```
   # Node.js example
   npm ci --production
   zip -r function.zip . -x "*.git*" "test/*"
   ```
   For Python:
   ```
   pip install -r requirements.txt -t package/
   cd package && zip -r ../function.zip . && cd ..
   zip -g function.zip lambda_function.py
   ```
2. If the ZIP exceeds 50 MB, upload to S3 first:
   ```
   aws s3 cp function.zip s3://<bucket>/deployments/function-<version>.zip
   ```

## CloudFormation Resources

- `AWS::Lambda::Function` — runtime, handler, code location, memory, timeout
- `AWS::Lambda::Alias` — point to a specific version for traffic shifting
- `AWS::Lambda::Version` — immutable snapshot of function code+config
- `AWS::ApiGateway::RestApi` or `AWS::ApiGatewayV2::Api` (if API-triggered)

## Deploy

Update the stack with the new code reference. CloudFormation will publish a new version if `AutoPublishAlias` is configured (SAM) or if you manage versions explicitly.

For quick code-only updates outside CloudFormation:
```
aws lambda update-function-code --function-name <name> --zip-file fileb://function.zip
```

## Verify

1. Invoke the function:
   ```
   aws lambda invoke --function-name <name> --payload '{"test": true}' response.json
   cat response.json
   ```
2. Check for errors in CloudWatch:
   ```
   aws logs tail /aws/lambda/<name> --since 5m
   ```

## Rollback

Switch the alias back to the previous version:
```
aws lambda update-alias --function-name <name> --name live --function-version <prev-version>
```
