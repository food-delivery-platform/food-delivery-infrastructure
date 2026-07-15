# restaurant-service + delivery-service infrastructure

This version contains only restaurant-service and delivery-service.

Temporary service runtime stacks:

- `delivery-service-runtime`
- `restaurant-service-runtime`

Temporary shared platform stack:

- `food-delivery-preview-platform`

## Automatic cleanup order

```text
delivery-service-runtime
→ restaurant-service-runtime
→ food-delivery-preview-platform
```

Deleting `delivery-service-runtime` and `restaurant-service-runtime` removes
their ECS Services and stops their Fargate tasks.

Deleting `food-delivery-preview-platform` removes:

- ECS cluster
- internal ALB
- listener and listener rules
- delivery and restaurant target groups
- temporary security groups
- temporary API Gateway routes and private integration
- the one-time EventBridge Scheduler schedule

The cleanup does not remove:

- `food-delivery-network`
- `food-delivery-auth-api`
- `food-delivery-cleanup`
- Cognito User Pool
- API Gateway itself
- VPC Link
- ECR repositories and images
- task-definition stacks and revisions
- databases

## Delivery service workflow outputs

The delivery-service repository should read:

- `ClusterName`
- `DeliveryTargetGroupArn`
- `DeliveryTaskSecurityGroupId`
- `SubnetIds`

Its temporary stack must be named:

```text
delivery-service-runtime
```

## Restaurant service workflow outputs

The restaurant-service repository should read:

- `ClusterName`
- `RestaurantTargetGroupArn`
- `RestaurantTaskSecurityGroupId`
- `SubnetIds`

Its temporary stack must be named:

```text
restaurant-service-runtime
```
