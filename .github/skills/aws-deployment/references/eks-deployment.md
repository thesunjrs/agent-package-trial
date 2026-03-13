# EKS (Kubernetes) Deployment

## Prerequisites

1. Update kubeconfig:
   ```
   aws eks update-kubeconfig --name <cluster-name> --region <region>
   ```
2. Verify connectivity:
   ```
   kubectl cluster-info
   ```

## Build & Push Image

Same as [ECS deployment](./ecs-deployment.md#build--push-image) — push to ECR.

## CloudFormation Resources (Cluster Infra)

- `AWS::EKS::Cluster`
- `AWS::EKS::Nodegroup` or Fargate profile
- `AWS::EC2::VPC`, subnets, security groups

Application manifests are typically managed via `kubectl` or Helm, not CloudFormation.

## Deploy Application

1. Update the image tag in the Kubernetes manifest or Helm values.
2. Apply:
   ```
   kubectl apply -f k8s/ --namespace <namespace>
   ```
   Or with Helm:
   ```
   helm upgrade <release> ./chart --set image.tag=<tag> --namespace <namespace>
   ```
3. Watch rollout:
   ```
   kubectl rollout status deployment/<name> --namespace <namespace> --timeout=300s
   ```

## Verify

1. Check pod status:
   ```
   kubectl get pods -l app=<name> --namespace <namespace>
   ```
2. Check service endpoint:
   ```
   kubectl get svc <name> --namespace <namespace> -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
   ```
3. Curl the health endpoint.

## Rollback

```
kubectl rollout undo deployment/<name> --namespace <namespace>
```

Or with Helm:
```
helm rollback <release> --namespace <namespace>
```
