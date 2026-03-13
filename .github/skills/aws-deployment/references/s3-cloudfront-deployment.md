# S3 + CloudFront Deployment

## Build

Run the project's static build step (e.g., `npm run build`). The output directory (e.g., `dist/`, `build/`) is what gets uploaded.

## CloudFormation Resources

- `AWS::S3::Bucket` — static hosting, OAC policy
- `AWS::CloudFront::Distribution` — origins, cache behaviors, custom domain
- `AWS::CloudFront::OriginAccessControl`
- `AWS::CertificateManager::Certificate` (if custom domain with HTTPS)

## Deploy

1. Sync build output to S3:
   ```
   aws s3 sync ./dist s3://<bucket-name> --delete
   ```
   `--delete` removes files from S3 that no longer exist locally.

2. Invalidate the CloudFront cache:
   ```
   aws cloudfront create-invalidation \
     --distribution-id <dist-id> \
     --paths "/*"
   ```
3. Wait for invalidation to complete:
   ```
   aws cloudfront wait invalidation-completed \
     --distribution-id <dist-id> \
     --id <invalidation-id>
   ```

## Verify

1. Check the CloudFront domain or custom domain in a browser.
2. Verify cache headers:
   ```
   curl -I https://<domain>/index.html
   ```
   Look for `X-Cache: Hit from cloudfront` after the invalidation propagates.

## Rollback

Re-sync the previous build output from a versioned S3 bucket or rebuild from the previous commit and re-deploy.

If S3 versioning is enabled:
```
aws s3api list-object-versions --bucket <bucket-name> --prefix index.html
```
Restore specific versions as needed.
