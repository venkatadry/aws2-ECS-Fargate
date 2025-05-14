# Deploying to ECS with GitLab CI/CD, Helm, and Terraform

Here's a comprehensive approach to setting up a GitLab CI/CD pipeline to deploy Docker images to your existing ECS cluster using Helm.

## Prerequisites
- Existing ECS cluster provisioned with Terraform
- Docker image built and stored in a container registry (ECR, GitLab Registry, etc.)
- Helm installed and configured
- AWS credentials configured for deployment

## 1. Helm Chart Structure for ECS

Create a Helm chart for your ECS service/task:

```
my-ecs-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── task-definition.yaml
│   ├── service.yaml
│   └── configmap.yaml (if needed)
```

### Example `templates/task-definition.yaml`:

```yaml
apiVersion: ecs.amazonaws.com/v1
kind: TaskDefinition
metadata:
  name: {{ .Values.taskDefinition.name }}
  namespace: {{ .Release.Namespace }}
spec:
  family: {{ .Values.taskDefinition.family }}
  networkMode: {{ .Values.taskDefinition.networkMode }}
  cpu: {{ .Values.taskDefinition.cpu }}
  memory: {{ .Values.taskDefinition.memory }}
  executionRoleArn: {{ .Values.taskDefinition.executionRoleArn }}
  taskRoleArn: {{ .Values.taskDefinition.taskRoleArn }}
  containerDefinitions:
    - name: {{ .Values.container.name }}
      image: {{ .Values.container.image }}
      portMappings:
        {{- toYaml .Values.container.portMappings | nindent 8 }}
      environment:
        {{- toYaml .Values.container.environment | nindent 8 }}
```

## 2. GitLab CI/CD Pipeline

Create a `.gitlab-ci.yml` file:

```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA
  HELM_CHART: "my-ecs-app"
  CLUSTER_NAME: "your-ecs-cluster-name"
  AWS_REGION: "us-east-1"

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

test:
  stage: test
  image: alpine
  script:
    # Add your tests here
    - echo "Running tests"

deploy:
  stage: deploy
  image: alpine/helm:latest
  before_script:
    - apk add --no-cache python3 py3-pip
    - pip3 install awscli
    - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
    - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
    - aws configure set region $AWS_REGION
  script:
    - helm upgrade --install $HELM_CHART ./$HELM_CHART
      --set container.image=$DOCKER_IMAGE
      --set taskDefinition.name="my-app-${CI_COMMIT_SHORT_SHA}"
      --namespace my-namespace
      --wait
```

## 3. Integrating with Existing Terraform

If you need to reference Terraform outputs in your Helm chart:

1. Export Terraform outputs to a file during your infrastructure pipeline
2. Use them in your Helm deployment:

```yaml
deploy:
  # ...
  script:
    - source terraform_outputs.env
    - helm upgrade --install $HELM_CHART ./$HELM_CHART
      --set container.image=$DOCKER_IMAGE
      --set taskDefinition.executionRoleArn=$TF_OUTPUT_ECS_TASK_ROLE
      # other terraform outputs
```

## 4. Advanced Considerations

1. **Blue/Green Deployments**: Consider using AWS CodeDeploy with ECS for blue/green deployments
2. **Secrets Management**: Use AWS Secrets Manager or Parameter Store with `secrets` in your task definition
3. **Rollback Strategy**: Implement helm rollback or pipeline recovery steps
4. **Infrastructure Updates**: Extend your pipeline to update Terraform if needed

## 5. Alternative Approach: Terraform + Helm

If you prefer managing everything through Terraform:

```hcl
resource "helm_release" "ecs_app" {
  name       = "my-ecs-app"
  chart      = "./helm-charts/my-ecs-app"
  namespace  = "my-namespace"
  
  set {
    name  = "container.image"
    value = "${aws_ecr_repository.my_repo.repository_url}:${var.image_tag}"
  }
  
  set {
    name  = "taskDefinition.executionRoleArn"
    value = aws_iam_role.ecs_task_execution_role.arn
  }
}
```

Would you like me to elaborate on any specific part of this setup?
