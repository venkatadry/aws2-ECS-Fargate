
https://medium.com/@alimaisam/mastering-aws-ecs-fargate-troubleshooting-deployment-pitfalls-8e5b77621316
https://medium.com/vrt-digital-studio/troubleshooting-ecs-task-failed-to-start-6b5719b11ded
https://medium.com/@gwenleigh
https://medium.com/@gwenleigh/week-6-fargate-troubleshooting-frontend-health-check-3-aws-ecs-internal-check-8cfe0997120a
https://medium.com/@gwenleigh/week-6-ecs-fargate-atlas-of-errors-507302d4fff8 - good
https://medium.com/@gwenleigh/week-7-troubleshoot-fargate-containers-keep-deregistering-633ca1b34b03

https://medium.com/@sharma.naman/common-ecs-issues-and-their-solutions-c031f00e0283 - good

https://mihirpopat.medium.com/mastering-aws-ecs-scenario-based-questions-you-must-know-f999cb000551

# ECS (Elastic Container Service) Interview Questions and Answers

## Basic Level Questions

### 1. What is Amazon ECS?
**Answer:** Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that makes it easy to deploy, manage, and scale containerized applications. It supports Docker containers and allows you to run applications on a managed cluster of Amazon EC2 instances or using AWS Fargate (serverless compute for containers).

### 2. What are the main components of ECS?
**Answer:**
- **Clusters**: Logical grouping of tasks or services
- **Task Definitions**: Blueprint for your application (container images, CPU/memory, networking, etc.)
- **Tasks**: Instantiation of a task definition (running containers)
- **Services**: Maintains a specified number of tasks running simultaneously
- **Container Instances**: EC2 instances running the ECS agent (when using EC2 launch type)

### 3. What are the two launch types in ECS?
**Answer:**
1. **EC2 launch type**: You manage the EC2 instances in your cluster
2. **Fargate launch type**: Serverless option where AWS manages the infrastructure

### 4. What is an ECS Task Definition?
**Answer:** A Task Definition is a JSON file that describes one or more containers that form your application. It specifies parameters like:
- Which container images to use
- CPU and memory requirements
- Networking mode
- Logging configuration
- Environment variables
- Storage and volume requirements

### 5. How does ECS differ from EKS?
**Answer:**
- **ECS** is AWS's proprietary container orchestration service with tight AWS integration
- **EKS** (Elastic Kubernetes Service) is AWS's managed Kubernetes service
- ECS is simpler to set up and manage but has fewer features than Kubernetes
- EKS follows Kubernetes standards and can be more portable across clouds

## Intermediate Level Questions

### 6. What is the difference between ECS Tasks and Services?
**Answer:**
- **Tasks** are single running instances of a Task Definition. They can be run standalone (task) or as part of a Service.
- **Services** are used to maintain a desired number of tasks running simultaneously. They provide features like load balancing, scaling, and automatic recovery if a task fails.

### 7. How does ECS integrate with other AWS services?
**Answer:** ECS integrates with:
- **ELB/ALB/NLB** for load balancing
- **IAM** for permissions and roles
- **CloudWatch** for logging and monitoring
- **VPC** for networking
- **ECR** for container image storage
- **Secrets Manager/Parameter Store** for sensitive data
- **EventBridge** for event-based triggers

### 8. What are the different networking modes available in ECS?
**Answer:**
1. **Bridge mode**: Default Docker networking (NAT)
2. **Host mode**: Containers share host's network stack
3. **awsvpc mode**: Each task gets its own elastic network interface (ENI) with private IP
4. **None**: No networking configured

### 9. How do you monitor ECS containers?
**Answer:**
- **CloudWatch Logs**: Container logs can be sent to CloudWatch
- **CloudWatch Metrics**: CPU, memory, network metrics
- **ECS Event Stream**: For task state changes
- **Container Insights**: Enhanced monitoring for ECS resources
- **Third-party tools**: Datadog, New Relic, etc. via sidecar containers

### 10. What is the role of the ECS agent?
**Answer:** The ECS agent is a container that runs on each EC2 instance in an ECS cluster (EC2 launch type). Its responsibilities include:
- Communicating with the ECS service
- Starting/stopping tasks
- Reporting task status and resource utilization
- Sending container metrics to CloudWatch

## Advanced Level Questions

### 11. How does ECS handle service discovery?
**Answer:** ECS integrates with:
1. **Cloud Map**: AWS's service discovery service that maintains up-to-date DNS records for services
2. **Internal ALBs/NLBs**: Services can be registered with load balancers
3. **Third-party solutions**: Consul, etc. via custom solutions

### 12. Explain how auto-scaling works in ECS.
**Answer:** ECS supports two types of scaling:
1. **Service Auto Scaling**: Scales the number of tasks in a service based on CloudWatch metrics or custom metrics
   - Target Tracking: Maintain a specific metric value (e.g., CPU utilization)
   - Step Scaling: Scale based on defined thresholds
   - Scheduled Scaling: Predictable load changes

2. **Cluster Auto Scaling (EC2 launch type)**: Scales the number of EC2 instances in the cluster based on resource needs
   - Uses EC2 Auto Scaling Groups
   - Can scale based on CloudWatch metrics or ECS capacity providers

### 13. What are ECS Capacity Providers and how do they work?
**Answer:** Capacity Providers are a flexible way to manage infrastructure for ECS tasks. They:
- Define how tasks are distributed across infrastructure
- Can be associated with either EC2 Auto Scaling Groups or Fargate
- Include a strategy (FARGATE, FARGATE_SPOT, or EC2-based)
- Enable features like managed scaling and managed termination protection
- Allow prioritization of task placement across different capacity providers

### 14. How would you implement blue/green deployments in ECS?
**Answer:** Common approaches include:
1. **Using AWS CodeDeploy**:
   - Create a new task definition revision
   - CodeDeploy shifts traffic between original (blue) and new (green) environments
   - Supports canary, linear, and all-at-once deployment styles

2. **Using ALB with two target groups**:
   - Create two services pointing to different task definitions
   - Shift ALB traffic weights gradually between target groups
   - Delete old service after verification

3. **Using Route 53 weighted routing** between two ALBs

### 15. What are some best practices for securing ECS workloads?
**Answer:**
- Use IAM roles for tasks instead of hardcoding credentials
- Store secrets in Secrets Manager or Parameter Store
- Enable ECR image scanning for vulnerabilities
- Use network segmentation with security groups and NACLs
- Implement task definitions with minimal required permissions
- Enable VPC Flow Logs for network traffic monitoring
- Use Fargate for improved isolation (no shared kernel)
- Regularly update container images and underlying AMIs (for EC2 launch type)

### 16. How does ECS handle persistent storage?
**Answer:** Options include:
1. **EFS Volumes**:
   - Can be mounted by multiple tasks simultaneously
   - Persistent beyond task lifecycle
   - Works with both EC2 and Fargate launch types

2. **EBS Volumes**:
   - Only available with EC2 launch type
   - Tied to specific EC2 instances
   - Not recommended for most ECS use cases

3. **Docker volumes**:
   - Ephemeral storage on the host (EC2 launch type)
   - Lost when task stops

4. **S3**: For object storage via application-level integration

### 17. Explain how spot instances can be used with ECS.
**Answer:** There are two main approaches:
1. **EC2 launch type with Spot Instances**:
   - Configure the Auto Scaling Group to use Spot Instances
   - Use capacity providers with Spot Instances
   - Implement task placement strategies to handle interruptions

2. **Fargate Spot**:
   - Up to 70% cost savings compared to Fargate
   - AWS manages the underlying Spot infrastructure
   - Tasks can be interrupted with 2-minute warning
   - Best for fault-tolerant, interruptible workloads

### 18. What are ECS Exec and how do you use it?
**Answer:** ECS Exec enables interactive shell access to running containers:
1. Requirements:
   - AWS CLI version 2
   - Session Manager plugin installed
   - Tasks must use platform version 1.4.0 or later
   - Task execution role needs additional permissions

2. Usage:
   ```bash
   aws ecs execute-command --cluster cluster-name --task task-id --container container-name --interactive --command "/bin/sh"
   ```
3. Benefits:
   - Secure access without opening inbound ports
   - Audit logging via CloudTrail
   - No SSH keys or bastion hosts needed

### 19. How would you troubleshoot a task that fails to start?
**Answer:**
1. Check ECS event logs for the task
2. Examine stopped task reason (describe-tasks API)
3. Review container logs in CloudWatch
4. Verify:
   - Task definition parameters (CPU/memory)
   - IAM permissions for task role and execution role
   - Network connectivity (security groups, subnets)
   - Image availability in ECR
   - Resource availability in cluster
5. Use ECS Exec to inspect running containers
6. Check service quotas if hitting limits

### 20. What are some advanced task placement strategies in ECS?
**Answer:**
1. **Built-in strategies**:
   - Binpack: Place tasks to minimize number of instances
   - Spread: Distribute tasks evenly across instances/zones
   - Random: Simple random placement

2. **Custom strategies** using multiple attributes:
   - Instance type
   - Availability Zone
   - Custom attributes
   - Combine with spread/binpack for sophisticated placement

3. **Constraints**:
   - Distinct Instance: One task per instance
   - Member Of: Place in specific cluster/attribute

4. **Capacity Provider strategies**:
   - Mix of On-Demand and Spot instances
   - Prioritize certain capacity providers

These questions cover a wide range of ECS concepts from fundamental to advanced, helping you prepare for interviews at various levels of expertise.



**Troubleshooting**
# Compute resource definitions are nested in the wrong attribute

```
{
    "family": "backend-flask",
    "executionRoleArn": "arn:aws:iam::501753804709:role/CruddurServiceExecutionRole",
    "taskRoleArn": "arn:aws:iam::501753804709:role/CruddurTaskRole",
    "networkMode": "awsvpc",
    "containerDefinitions": [
      {
        "name": "backend-flask",
        "image": "501753804709.dkr.ecr.us-east-1.amazonaws.com/backend-flask",
        "cpu": 256,         <--------- these attributes shouldn't be nested
        "memory": 512,      <---------
        "essential": true,
        "portMappings": [
```
# Correctly updated
```
# Correctly updated
{
    "family": "backend-flask",
    "executionRoleArn": "arn:aws:iam::501753804709:role/CruddurServiceExecutionRole",
    "taskRoleArn": "arn:aws:iam::501753804709:role/CruddurTaskRole",
    "networkMode": "awsvpc",
    "cpu": 256,       <------------- Define fargate server resource
    "memory": 512,    <-------------
    "containerDefinitions": [
      {
        "name": "backend-flask",
        "image": "501753804709.dkr.ecr.us-east-1.amazonaws.com/backend-flask",
        "essential": true,
        "portMappings": [
```

**Issue2**
Backend Health check 1 (local)

Findings & leads
I found that the line response = urllib.request.urlopen(HEALTH_CHECK_URL) is the problem. The following are the url:error_message pairs.

HTTP Error 500: INTERNAL SERVER ERROR from "http ://localhost:4567/api/health-check"
HTTP Error 404: Not Foun from "https: //4567-... .gitpod.io/api/health-check"

The whole point of running a health check is to verify whether the server is running and if the api routing is working. If you are getting the classic 500 internal server error and 404 not found error, it is most likely there is a problem with the routing.

1) In my case, the cause of the problem was that I forgot to add the health check route to my flask’s app.py.
# add the route for health check in app.py
@app.route('api/health-check')
def health_check():
  return {'success': True}, 200

**Issue3**
  An error occurred (InvalidParameterException) 
    when calling the CreateService operation: 
    The target group with targetGroupArn  # Launching the backend container failed all of a sudden.
    arn:aws:elasticloadbalancing:REGION:AWS_ACCOUNT_ID:targetgroup/cruddur-backend-flask-tg/da...
    does not have an associated load balancer. 

    It turns out, I missed adding a rule that routes api.domain.com to the backend-flask target group.
Solution
You have to add “a rule that forwards api. traffic to the backend” to the HTTPS:443 listener in your Application Load Balancer.
Compare the following listener settings, with an extra attention to the Default action and Rules rules columns.

**Issue4**
WORKDIR /frontend-react-js  
COPY . ./frontend-react-js   <----- this line must precede WORKDIR
RUN npm install
RUN npm run build

The docker command WORKDIR only “defines” the working directory, instead of “creating” one.
— COPY does create a directory.
— Hence, COPY first then WORKDIR.
# Dockerfile.prod
...

COPY . ./frontend-react-js   <--- Switch the order.
WORKDIR /frontend-react-js   <--- 

**Issue5**
