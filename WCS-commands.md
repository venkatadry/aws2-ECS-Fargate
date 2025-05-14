Here are some commonly used **AWS ECS (Elastic Container Service)** commands for managing clusters, tasks, services, and container instances:

---

### **1. Cluster Management**
#### List all ECS clusters:
```bash
aws ecs list-clusters
```

#### Describe a specific cluster:
```bash
aws ecs describe-clusters --clusters <cluster-name>
```

#### Delete a cluster:
```bash
aws ecs delete-cluster --cluster <cluster-name>
```

---

### **2. Task Definitions**
#### List task definitions:
```bash
aws ecs list-task-definitions
```

#### Describe a task definition:
```bash
aws ecs describe-task-definition --task-definition <task-definition-family:revision>
```

#### Deregister a task definition:
```bash
aws ecs deregister-task-definition --task-definition <task-definition-arn>
```

#### Register a new task definition (from JSON file):
```bash
aws ecs register-task-definition --cli-input-json file://<path-to-task-definition.json>
```

---

### **3. Tasks**
#### List running tasks in a cluster:
```bash
aws ecs list-tasks --cluster <cluster-name>
```

#### Describe a task:
```bash
aws ecs describe-tasks --cluster <cluster-name> --tasks <task-id>
```

#### Stop a running task:
```bash
aws ecs stop-task --cluster <cluster-name> --task <task-id>
```

#### Run a new task:
```bash
aws ecs run-task --cluster <cluster-name> --task-definition <task-definition-family:revision>
```

---

### **4. Services**
#### List services in a cluster:
```bash
aws ecs list-services --cluster <cluster-name>
```

#### Describe a service:
```bash
aws ecs describe-services --cluster <cluster-name> --services <service-name>
```

#### Update a service (e.g., change task definition or desired count):
```bash
aws ecs update-service --cluster <cluster-name> --service <service-name> --task-definition <new-task-definition> --desired-count <count>
```

#### Scale a service:
```bash
aws ecs update-service --cluster <cluster-name> --service <service-name> --desired-count <count>
```

#### Delete a service:
```bash
aws ecs delete-service --cluster <cluster-name> --service <service-name> --force
```

---

### **5. Container Instances**
#### List container instances in a cluster:
```bash
aws ecs list-container-instances --cluster <cluster-name>
```

#### Describe a container instance:
```bash
aws ecs describe-container-instances --cluster <cluster-name> --container-instances <container-instance-id>
```

#### Deregister a container instance:
```bash
aws ecs deregister-container-instance --cluster <cluster-name> --container-instance <container-instance-id> --force
```

---

### **6. ECR (Elastic Container Registry) Integration**
#### Login to ECR:
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

#### Push an image to ECR:
```bash
docker tag <local-image>:<tag> <account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>:<tag>
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>:<tag>
```

#### List ECR repositories:
```bash
aws ecr describe-repositories
```

---

### **7. Exec into a Running Task (ECS Exec)**
#### Enable ECS Exec for a task:
```bash
aws ecs update-service --cluster <cluster-name> --service <service-name> --enable-execute-command --force-new-deployment
```

#### Start a shell session inside a running container:
```bash
aws ecs execute-command --cluster <cluster-name> --task <task-id> --container <container-name> --interactive --command "/bin/sh"
```

---

### **Additional Useful Commands**
#### Check ECS agent version on container instances:
```bash
aws ecs describe-container-instances --cluster <cluster-name> --container-instances <instance-id> --query 'containerInstances[].versionInfo'
```

#### Get task protection status:
```bash
aws ecs describe-task-protection --cluster <cluster-name> --tasks <task-id>
```

---

### **Tips**
- Replace `<cluster-name>`, `<task-id>`, `<service-name>`, etc., with your actual values.
- Use `--query` for filtering JSON responses (e.g., `--query 'services[].serviceName'`).
- Use `--output table/json/text` for different output formats.

Let me know if you need help with a specific ECS scenario! 🚀

#################################################SSM ################################################################################################################

In **Amazon ECS (Elastic Container Service)**, **SSM (AWS Systems Manager)** is commonly used for **managing containers and tasks** without requiring SSH access. Here are the key uses of SSM in ECS:

### 1. **Executing Commands Inside ECS Tasks (Without SSH)**
   - **ECS Exec** (powered by SSM) allows you to run commands inside a running ECS task (container) without needing SSH.
   - Useful for debugging, troubleshooting, or running administrative commands.
   - Example:
     ```bash
     aws ecs execute-command \
       --cluster my-cluster \
       --task my-task-id \
       --container my-container \
       --command "/bin/sh" \
       --interactive
     ```

### 2. **Secure Remote Access to Containers**
   - SSM Session Manager provides a secure way to access ECS tasks without exposing them to the internet.
   - No need to open inbound ports (like SSH port 22), improving security.

### 3. **Automating Management Tasks**
   - SSM Run Command can execute predefined scripts across multiple ECS tasks.
   - Useful for batch operations like log rotation, updates, or configuration changes.

### 4. **Retrieving Logs & Metrics**
   - SSM can help fetch logs from ECS tasks if they are integrated with CloudWatch.
   - Can be used alongside AWS CloudWatch for monitoring.

### 5. **Managing ECS Instances (EC2 Launch Type)**
   - If using the **EC2 launch type**, SSM can manage the underlying EC2 instances (patching, updates, etc.).

### **How to Enable SSM for ECS?**
1. **Ensure ECS Tasks Have SSM Permissions**  
   - Attach an IAM policy with `ssm:StartSession` permissions to the ECS task role.
   - Example policy:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": [
             "ssmmessages:CreateControlChannel",
             "ssmmessages:CreateDataChannel",
             "ssmmessages:OpenControlChannel",
             "ssmmessages:OpenDataChannel"
           ],
           "Resource": "*"
         }
       ]
     }
     ```

2. **Enable ECS Exec When Starting a Task**  
   - Use the `--enable-execute-command` flag:
     ```bash
     aws ecs run-task \
       --cluster my-cluster \
       --task-definition my-task-definition \
       --enable-execute-command
     ```

### **Conclusion**
SSM in ECS is primarily used for **secure command execution, debugging, and automation**, reducing reliance on SSH and improving security. It's especially useful in **Fargate** where direct SSH access is not possible.

Would you like a specific example of using SSM with ECS?
