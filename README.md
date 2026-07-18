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
any ECS Services and Fargate tasks created by those runtime stacks.

The preview platform creates a Fargate-ready ECS cluster. Fargate tasks do not
appear as EC2 container instances in the ECS cluster. If the GitHub secret
`RESTAURANT_SERVICE_IMAGE_URI` is set to a full ECR image URI, the platform also
creates the restaurant-service task definition, ECS Service, and one running
Fargate task.

Deleting `food-delivery-preview-platform` removes:

- ECS cluster
- internal ALB
- listener and listener rules
- delivery and restaurant target groups
- temporary security groups
- temporary API Gateway routes and private integration
- optional restaurant-service ECS Service, task definition, task execution role,
  and log group
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
- `AssignPublicIp` (`ENABLED`, because the exported subnets are public)
- `DefaultCapacityProvider` (`FARGATE`)

Its temporary stack must be named:

```text
delivery-service-runtime
```

## Restaurant service image

To run restaurant-service as part of the platform deployment, set the GitHub
secret `RESTAURANT_SERVICE_IMAGE_URI` to the full image URI:

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/restaurant-service:latest
```

Use the image URI, not the ECR repository ARN. The repository ARN identifies the
repository, but ECS task definitions need a pullable image reference including a
tag or digest.

The platform still exports these values if restaurant-service is deployed by a
separate runtime stack:

- `ClusterName`
- `RestaurantTargetGroupArn`
- `RestaurantTaskSecurityGroupId`
- `SubnetIds`
- `AssignPublicIp` (`ENABLED`, because the exported subnets are public)
- `DefaultCapacityProvider` (`FARGATE`)
