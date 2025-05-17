Created policy 
**Policy Name: ECSDeploymentPermissions**
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ECSDeploymentPermissions",
      "Effect": "Allow",
      "Action": [
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService",
        "ecs:RunTask",
        "ecs:Describe*",
        "ecs:List*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowPassRoleForECS",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::920373005946:role/ecsTaskExecutionRole"
    }
  ]
}

```
```
Elastic Container Service
Full: List Limited: Read, Write
All resources
None

IAM
Limited: Write
RoleName| string like |ecsTaskExecutionRole
None
```

Attached the policy ECSDeploymentPermissions to test-user
