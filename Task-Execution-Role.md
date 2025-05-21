# ECS Execution Role

An ECS Execution Role is an AWS Identity and Access Management (IAM) role that Amazon Elastic Container Service (ECS) uses to make API calls on your behalf to other AWS services.

## Key Functions

The ECS Execution Role is primarily used for:
- Pulling container images from Amazon ECR (Elastic Container Registry)
- Sending container logs to CloudWatch Logs
- Using AWS Secrets Manager or Systems Manager Parameter Store for sensitive data
- Accessing other AWS services that your containers need

## Differences from Task Role

It's important to distinguish between:
- **Execution Role**: Used by the ECS agent to manage tasks (pulling images, etc.)
- **Task Role**: Assigned to the running containers to access AWS services during execution

## Creating an Execution Role

You can create an execution role with the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

## Best Practices

1. Use the principle of least privilege - only grant necessary permissions
2. Separate execution roles for different environments (dev, staging, prod)
3. Regularly review and update role permissions
4. Use AWS-managed policies like `AmazonECSTaskExecutionRolePolicy` when appropriate

## Common Issues

- "NoCredentialProviders" error often indicates execution role problems
- "AccessDenied" when pulling images usually means missing ECR permissions
- Ensure the role is properly assigned to the task definition

Would you like more specific information about any aspect of ECS execution roles?
