# Food-delivery infrastructure with EventBridge cleanup

## Runtime request path

```text
Frontend
→ API Gateway HTTP API
→ Cognito JWT authorizer
→ API Gateway VPC Link
→ internal ALB
├── /api/orders* → order-service
└── /api/restaurants* → restaurant-service
```

## Cleanup path

```text
GitHub Actions
→ deploys permanent infrastructure
→ deploys the temporary preview platform
→ creates AWS::Scheduler::Schedule with at(<UTC timestamp>)
→ finishes immediately

At the requested time:

EventBridge Scheduler
→ starts Step Functions
→ deletes order-service-runtime
→ waits until it is deleted
→ deletes restaurant-service-runtime
→ waits until it is deleted
→ deletes food-delivery-preview-platform
→ waits until it is deleted
```

GitHub Actions no longer runs `sleep 3600` and does not remain occupied for the lifetime of the preview environment.

## Stack lifecycle

Permanent:

- `food-delivery-network`
- `food-delivery-auth-api`
- `food-delivery-cleanup`
- ECR and task-definition stacks belonging to services

Temporary:

- `food-delivery-preview-platform`
- `order-service-runtime`
- `restaurant-service-runtime`

The EventBridge schedule belongs to `food-delivery-preview-platform`. Once the schedule starts the permanent cleanup state machine, the state machine deletes the runtime stacks and finally deletes the platform stack together with the schedule.

## Permanent cleanup stack

`infrastructure/cleanup.yml` creates:

- `food-delivery-preview-stack-cleanup` Lambda
- a Standard Step Functions state machine
- Lambda execution role
- Step Functions execution role
- EventBridge Scheduler target role

The Lambda accepts only these stack names:

- `order-service-runtime`
- `restaurant-service-runtime`
- `food-delivery-preview-platform`

## Deploy

Run:

```text
Actions
→ Deploy API preview with EventBridge cleanup
→ Run workflow
```

The input `lifetime_minutes` controls when EventBridge Scheduler starts cleanup.

The workflow calculates an expression such as:

```text
at(2026-07-15T20:30:00)
```

and passes it to `preview-platform.yml`.

## Manual early cleanup

Run:

```text
Actions
→ Start AWS cleanup now
→ Run workflow
```

This workflow does not delete stacks itself. It only starts the same permanent Step Functions cleanup workflow that EventBridge Scheduler uses.

## Repository files

```text
food-delivery-infrastructure/
├── infrastructure/
│   ├── network.yml
│   ├── auth-api.yml
│   ├── cleanup.yml
│   └── preview-platform.yml
├── .github/workflows/
│   ├── deploy-eventbridge-preview.yml
│   └── start-cleanup-now.yml
├── iam/
│   └── github-actions-eventbridge-platform-policy.json
└── README.md
```
