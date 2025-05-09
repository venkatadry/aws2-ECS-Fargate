***section 1.1***
### **Notes: Welcome to 2025 AWS ECS Updates**  

#### **Introduction**  
- The session covers new AWS ECS features and components released in the last five years.  
- Few updates were made to the course because AWS ECS has remained relatively stable.  
- The goal is to help learners scale up their ECS skills.  

---  

### **Docker Recap (2025)**  
- **Docker Basics Remain Unchanged:**  
  - Docker daemon, Docker client, `docker build`, `docker run`, etc., function the same way.  
  - No major changes in core Docker commands.  
- **Docker Registry:**  
  - Docker Hub and AWS ECR (Elastic Container Registry) are still the primary options.  
- **Dockerfile:**  
  - No significant changes; syntax and functionality remain the same.  

#### **Docker Architecture Recap**  
1. **Docker Daemon:** Runs on local machines, EC2, or any virtual server.  
2. **Docker Client:** Used to send commands (`docker build`, `docker run`).  
3. **Image Creation:**  
   - `docker build` → Daemon creates an image from Dockerfile.  
   - `docker push` → Uploads the image to a registry (Docker Hub/ECR).  
4. **Container Execution:**  
   - `docker run` → Starts containers (single or multiple).  
   - `docker pull` → Fetches images from a registry.  

#### **Docker Networking & Storage**  
- **Networking Modes:**  
  - Bridge, Host, and None networks remain unchanged.  
- **Storage Options:**  
  - **Docker Volume:** Managed by Docker daemon (host file system).  
  - **Bind Mount:** Manually mapped host file system to container.  

---  

### **AWS ECS New Features & Services**  

#### **1. AWS App Runner**  
- **Purpose:** Simplifies deploying containerized apps without managing infrastructure.  
- **Use Cases:**  
  - Developers who don’t want to learn ECS/EKS.  
  - Focus on application logic rather than orchestration.  
- **Two Deployment Methods:**  
  1. **Source Code Deployment:** Provide code (e.g., Python Flask app), and App Runner handles the rest.  
  2. **Container Image Deployment:** Pre-built images can be deployed directly.  
- **Fully Managed:** Handles cluster creation, networking, and scaling.  

#### **2. AWS Copilot (CLI Tool)**  
- **Purpose:** Simplifies deploying containerized apps on ECS/App Runner.  
- **Key Commands:**  
  - `copilot init` → Initializes an application.  
  - `copilot env init` → Sets up the environment.  
  - Supports different service types (web services, backend services, scheduled jobs).  
- **Advantage:** Easier syntax than AWS CLI for ECS deployments.  

#### **3. AWS ECS Anywhere**  
- **Key Feature:** Run ECS tasks **outside AWS** (on-premise, other clouds, local machines).  
- **Launch Type:** **External** (for self-managed compute).  
- **Supported Environments:**  
  - On-premise physical servers/virtual machines.  
  - Virtual machines on other cloud providers.  
- **Agents Required:**  
  1. **AWS SSM Agent:** Manages connection between on-premise and ECS **service**.  
  2. **ECS Agent:** Connects to the ECS **cluster**.  
- **Key Benefits:**  
  - Tasks continue running even if connection to AWS is lost.  
  - Works seamlessly with AWS services (RDS, CloudWatch, DynamoDB).  
  - Supports IAM roles for tasks running outside AWS.  

---  

### **Summary of Key Takeaways**  
1. **Docker remains stable** with minimal changes in core functionality.  
2. **AWS App Runner** simplifies container deployment without infrastructure management.  
3. **AWS Copilot** provides an easier CLI for ECS/App Runner deployments.  
4. **ECS Anywhere** is a game-changer, allowing ECS to run on any infrastructure (on-premise, multi-cloud).  

**section1.2**
# **Notes on ECS Clusters and Capacity Providers**

## **1. Introduction to ECS Clusters**
- **ECS Clusters** are logical groupings of containerized applications running on AWS.
- They host the containers that run our applications.
- AWS ECS supports multiple **launch types**:
  - **EC2 Launch Type**: Uses EC2 instances to run containers.
  - **Fargate Launch Type**: Serverless, no need to manage EC2 instances.
  - **External Launch Type**: For on-premises or non-AWS infrastructure.

## **2. Key Components of ECS Clusters**
### **A. Tasks and Task Definitions**
- **Task Definition**: A blueprint that defines how containers should run (e.g., CPU, memory, networking).
- **Task**: A running instance of a task definition (one or more containers).
- Tasks run on **EC2 instances (container instances)** or **Fargate**.

### **B. Capacity Providers**
- **Purpose**: Automatically scales infrastructure (EC2 instances or Fargate) based on task demand.
- **Types of Capacity Providers**:
  1. **Auto Scaling Group (ASG)** – Used for **EC2 launch type**.
     - Scales EC2 instances up/down based on demand.
  2. **Fargate & Fargate Spot** – Used for **serverless** deployments.
     - Fargate Spot is cheaper but can be interrupted (like EC2 Spot).
  3. **Not Supported for External Launch Type** (since AWS doesn’t control on-prem resources).

- A **single ECS cluster** can have **multiple capacity providers** (e.g., mix of EC2 and Fargate).

### **C. How Capacity Providers Work**
- If an ECS cluster has resources for **4 tasks** but needs to run **6 tasks**, it scales up using:
  - **ASG (for EC2)**: Spins up new EC2 instances.
  - **Fargate**: Automatically provisions compute.

## **3. Hands-on Steps to Create an ECS Cluster (EC2 Launch Type)**
### **Prerequisites**
1. **IAM Role for EC2 Instances**:
   - Role: `AmazonEC2ContainerServiceforEC2Role`.
   - Allows EC2 instances to communicate with ECS.
2. **VPC Setup**:
   - Create a **VPC** (e.g., CIDR `10.0.0.0/16`).
   - Create **public subnets** (one per AZ).
   - Attach an **Internet Gateway**.
   - Configure **route tables** for public access.
3. **Security Group**:
   - Allow **all traffic** (for simplicity; restrict in production).

### **Creating the ECS Cluster**
1. **Go to ECS Console** → **Clusters** → **Create Cluster**.
2. **Cluster Configuration**:
   - **Cluster Name**: e.g., `my-ecs-cluster`.
   - **Infrastructure**: **EC2** (not Fargate).
   - **Auto Scaling Group (ASG)**:
     - **Provisioning Model**: On-Demand or Spot.
     - **Instance Type**: e.g., `t3.medium` (2 vCPU, 4GB RAM).
     - **Minimum/Maximum Instances**: e.g., Min=1, Max=5.
   - **Networking**:
     - Select the **VPC** and **public subnets**.
     - Attach the **security group**.
     - Enable **public IP** (for SSH access).
   - **Monitoring**: Enable **Container Insights** for observability.
3. **Review & Create** → AWS provisions the cluster via **CloudFormation**.

### **Post-Creation Observations**
- **Cluster Dashboard** shows:
  - **Services** (none initially).
  - **Tasks** (none running yet).
  - **Capacity Providers**:
    - ASG (for EC2).
    - Fargate & Fargate Spot (if configured).
- **EC2 Instances**:
  - Visible in the **EC2 Console** (named `ECS Instance for [Cluster]`).
  - Managed by **Auto Scaling Group**.

## **4. Next Steps**
- **Task Definitions**: Define container specs (image, CPU, memory).
- **ECS Services**: Ensures desired tasks are always running.
- **Load Balancing**: Attach an ALB/NLB for traffic distribution.

## **Key Takeaways**
- **ECS Clusters** manage containers using **EC2, Fargate, or external** resources.
- **Capacity Providers** (ASG, Fargate) handle **auto-scaling**.
- **Tasks** = Running containers; **Task Definitions** = Blueprint.
- **Best Practice**: Use private subnets + load balancers in production.

---

### **Additional Notes**
- **Fargate vs. EC2**:
  - **Fargate**: Simpler (no EC2 management) but slightly more expensive.
  - **EC2**: More control, cheaper for large workloads.
- **External Launch Type**: For hybrid cloud (e.g., on-prem Kubernetes).  

**section 1.3**
Here are the key notes from your explanation about the CloudFormation stack created during an ECS cluster launch:

---

### **ECS Cluster Creation via CloudFormation**
1. **Cluster Creation Process**  
   - When creating an ECS cluster (EC2 launch type), AWS automatically generates a CloudFormation stack.  
   - The stack name follows the format: `infra-<cluster-name>-IAM-ecs-cluster`.  

2. **Stack Components**  
   The CloudFormation stack creates the following resources in sequence:  
   - **ECS Cluster**: The core resource (e.g., `infra-X-cluster`).  
   - **Launch Template**: Defines EC2 instance configuration (e.g., AMI, instance type, IAM role).  
     - Required for the Auto Scaling Group (ASG) to launch EC2 instances.  
   - **Auto Scaling Group (ASG)**: Manages EC2 instances for the cluster (logical ID: `ECSASG`).  
   - **Capacity Provider**: Links the ASG to the ECS cluster (type: `ASG Capacity Provider`).  
     - For EC2 launch type: `ASG Capacity Provider` (CPP).  
     - For Fargate: `Fargate Capacity Provider`.  
   - **Cluster-Capacity Provider Association**: Binds the cluster to the ASG capacity provider.  

3. **Resource Names**  
   - Launch Template: Random suffix (e.g., `infra-X-cluster-xxxx`).  
   - ASG: Name includes `infra-X-cluster-<suffix>` (e.g., ends with `WFC`).  
   - Capacity Provider: Name includes `infra-X-cluster-<suffix>` (e.g., ends with `9ZF2`).  

4. **Verification in ECS Console**  
   - Navigate to **ECS Cluster → Infrastructure** tab:  
     - Confirms the ASG-linked capacity provider is active.  
     - Shows the ASG name (e.g., `infra-X-cluster-WFC`).  

5. **Stack Outputs & Parameters**  
   - **Outputs**: ECS cluster name.  
   - **Parameters**: Reflects user inputs during cluster creation (e.g., VPC, subnet, instance type).  

---

### **Important Notes**
- **Do NOT manually delete the CloudFormation stack**.  
  - Always delete the ECS cluster first → this automatically cleans up the stack.  
  - Manual stack deletion can orphan resources or break cluster tracking.  
- **Behind-the-Scenes Workflow**:  
  ```plaintext
  ECS Cluster → Launch Template → ASG → Capacity Provider → Cluster-Provider Association
  ```

---

### **Key Takeaways**
- CloudFormation automates ECS cluster infrastructure (ASG, capacity providers).  
- Capacity providers bridge ASG/Fargate to the ECS cluster.  
- Use the ECS console or CloudFormation events to track creation progress.  

Let me know if you'd like a diagram or further clarification!

**section 1.4**

### **AWS ECS Task Definition Notes**  

#### **Overview**  
- A **task definition** is a template for running containers in **AWS ECS**.  
- A **task** is an instance of a task definition (running containers).  
- A single task can run **one or more containers**.  

---

### **Task Definition Configuration Parameters**  

#### **1. Infrastructure Requirements**  
- **Task Family (Name & Version)** – Identifies the task definition.  
- **Launch Type** – Specifies how tasks run:  
  - **EC2** (on EC2 instances)  
  - **Fargate** (serverless)  
  - **External** (requires an ECS cluster)  
- **Operating System** – Linux (x86_64 or ARM) or Windows.  
- **Networking Mode** – Options:  
  - **AWS VPC** (recommended, dynamic port mapping)  
  - **Bridge** (manual host port mapping)  
  - **Host** (container port = host port)  
- **Task Size** – CPU & memory allocation.  
- **Task Role** – IAM role for containers to access AWS services (e.g., DynamoDB).  
- **Task Execution Role** – IAM role for ECS agent to pull images & manage tasks.  
- **Task Placement** – Controls how tasks are distributed across EC2 instances.  
- **Fault Injection** – Introduces errors to test task resilience (new feature).  

---

#### **2. Container Definition**  
- **Container Name** – Identifier for the container.  
- **Container Image** – Docker image location (ECR, Docker Hub, etc.).  
- **Essential Container** – If marked, stopping this container stops all others in the task.  
- **Port Mapping** – Varies by networking mode:  
  - **Bridge** → Specify both host & container ports.  
  - **Host** → Only container port (host port matches).  
  - **AWS VPC** → Only container port (host port dynamically assigned).  
- **Root File Access** – Controls read-only access to the container’s root filesystem.  
- **Resource Limits** – CPU, GPU, memory (soft/hard limits).  
- **Environment Variables** – Key-value pairs for configuration.  
- **Logging** – Supports:  
  - **CloudWatch**  
  - **Splunk**  
  - **Kinesis/Kinesis Firehose**  
  - **OpenSearch**  
  - **S3**  
  - **Third-party (Fluentd, Sumo Logic)**  
- **Restart Policy** – Defines container restart behavior (new).  
- **Health Check** – Configures interval, timeout, retries.  
- **Networking** – DNS settings, links between containers.  
- **Docker Config** – Entry point, command, working directory.  
- **Ulimit** – OS-level limits (CPU, file size, memory).  

---

#### **3. Storage & Volumes**  
- **Volume Types** – Options:  
  - **Bind Mount** (host ↔ container)  
  - **Docker Volume** (managed by Docker)  
  - **EFS (Elastic File System)** – Shared across containers/tasks.  
  - **FSX for Windows** – Used in Windows containers (not shared).  
- **EFS Use Case** – Multiple tasks/containers can read/write to the same EFS filesystem.  

---

#### **4. Monitoring & Deployment**  
- **Monitoring Tools** – Now supports:  
  - **Prometheus** (metrics)  
  - **AWS X-Ray** (tracing)  
- **Configuration Type** – Applies during:  
  - **Deployment** (container setup)  
  - **Task Creation**  

---

### **Key Takeaways**  
1. **Task Definition = Template**, **Task = Running Instance**.  
2. **EFS allows shared storage** across containers/tasks.  
3. **Essential Container** – If it fails, all containers in the task stop.  
4. **AWS VPC Networking** is preferred (dynamic port mapping).  
5. **New Features**: Fault injection, ulimit, expanded logging options.  

---
**Next Steps**:  
- Practice creating task definitions in AWS ECS.  
- Experiment with EFS for shared storage.  
- Explore monitoring with X-Ray/Prometheus.  

**section1.5**
Here are the summarized notes from your demo on task definition creation:

---

### **Task Definition Demo Summary**

#### **1. Objectives**
- Create an ECS task definition (without creating tasks).
- Implement a newsfeed app using ECS with:
  - 5 Python programs (containers).
  - 2 NoSQL databases: DynamoDB + ElastiCache.

#### **2. Steps Covered**
1. **Create ECR Repositories**  
   - Private repositories for each container image.
   - Used namespaces (e.g., `3am/`) to group repositories.
   - Repositories created:  
     - `add_news_articles`  
     - `breaking_news`  
     - `read_news`  
     - `read_news_articles`  
     - `report_news`  

2. **IAM Roles**  
   - **Task Role**: Grants permissions to containers (e.g., DynamoDB access).  
     - Created policy: Full DynamoDB access.  
     - Attached to role: `3IAM_ECS_Task_Policy_Role`.  
   - **Task Execution Role**: Allows ECS to manage containers.  
     - Used AWS-managed policy: `AmazonECSTaskExecutionRolePolicy`.  
     - Role name: `3IAM_ECS_Task_Execution_Role`.  

3. **Docker Setup**  
   - Created `Dockerfile` for each Python program:  
     - Base image: `python:3.12-slim`.  
     - Copied code + `requirements.txt` (generated via `pip freeze`).  
     - Installed dependencies with `pip3 install -r requirements.txt`.  
     - Exposed ports (e.g., `10002` for `add_news_articles`).  
   - Built and tagged images:  
     ```bash
     docker build -t add_news_articles .
     docker tag add_news_articles:latest <ECR_URI>/add_news_articles:v1
     ```  
   - Pushed images to ECR using `docker push`.  

4. **Task Definition Plan**  
   - Group containers into 3 task definitions:  
     - **Task 1**: `read_news` (essential) + `read_news_articles`.  
     - **Task 2**: `report_news` (essential) + `add_news_articles`.  
     - **Task 3**: `breaking_news` (standalone).  

5. **Next Steps (Not Shown in Demo)**  
   - Create task definitions in ECS console.  
   - Run tasks and verify:  
     - Check container status via `docker ps` on EC2 instances.  
     - Monitor tasks in ECS console.  

---

### **Key Commands**
- **ECR Push**:  
  ```bash
  aws ecr get-login-password | docker login --username AWS --password-stdin <ECR_URI>
  docker tag <image>:latest <ECR_URI>/<repo>:v1
  docker push <ECR_URI>/<repo>:v1
  ```

- **Docker Build**:  
  ```dockerfile
  FROM python:3.12-slim
  WORKDIR /app
  COPY . .
  RUN pip3 install -r requirements.txt
  EXPOSE <port>
  CMD ["python3", "<script>.py"]
  ```

---

### **Notes**
- **Namespaces**: Optional but useful for organizing repos (e.g., by team).  
- **IAM Roles**: Critical for permissions (task role = app permissions; execution role = ECS permissions).  
- **Port Mapping**: Dynamic in AWS VPC mode (host port auto-assigned).  

Let me know if you'd like to expand on any section!
