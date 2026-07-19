# Order-service Payments Infrastructure

This stack contains permanent AWS infrastructure for the order-service payment flow.
It must not be added to `infrastructure/preview-platform.yml`, because the preview
stack is temporary.

## Stack

- Stack name: `food-delivery-order-payments`
- Template: `infrastructure/order-service-payments.yml`
- Region: `us-east-1`
- EventBridge bus: `food-delivery-orders`
- PayPal webhook path: `/api/payments/paypal/webhook`
- Payment state machine: `payment-confirmation-sm`
- Step Functions role: `sfn-order-service-role`
- Lambda role: `lambda-order-service-role`

## Deployment Order

1. Deploy this stack first with `EnablePaypalWebhookRoute=false`.

   This creates the permanent EventBridge bus, the payment Step Functions state
   machine, order-service IAM roles, and inline policies for those roles. It does
   not create the API Gateway integration, route, or Lambda permission, so it can
   run before the `paypal_webhook` Lambda exists.

   If `sfn-order-service-role` and `lambda-order-service-role` are already
   managed by another stack, deploy with `CreateOrderServiceIamRoles=false`.

2. Deploy `order-service`.

   The service deployment should create the Lambda functions used by the payment
   flow, including `paypal_webhook`, `verify_payment`, `mark_payment_result`, and
   `publish_order_event`.

3. Deploy this stack again with `EnablePaypalWebhookRoute=true`.

   This adds the public `POST /api/payments/paypal/webhook` route to the existing
   HTTP API Gateway and grants API Gateway permission to invoke only the
   `paypal_webhook` Lambda for that route.

## GitHub Actions

Use the `Deploy order-service payments` workflow:

1. Run it with `enable_webhook_route` disabled for the first deployment.
   Leave `create_iam_roles` enabled unless those roles are managed elsewhere.
2. Deploy `order-service`.
3. Run it again with `enable_webhook_route` enabled.

The workflow validates the CloudFormation template, deploys
`food-delivery-order-payments` with `CAPABILITY_NAMED_IAM`, and prints the stack
outputs at the end.
