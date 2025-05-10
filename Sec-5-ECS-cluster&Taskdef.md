***################################################################################***

**31 ECS Architecture and workflow**

**################################################################################**
# **Elastic Container Service (ECS) - Notes**  

## **Introduction**  
- ECS is a **container orchestrator** provided by AWS.  
- It supports **Docker containers** only.  
- Works best for **microservices architecture**.  
- Integrates with **ELB, Auto Scaling, CI/CD tools** (AWS-native & third-party).  

---  

## **ECS Components**  
### **1. ECS Cluster**  
- A **pool of EC2 instances** (called **container instances**).  
- Uses **ECS-optimized AMIs** (pre-installed with Docker daemon & ECS agent).  
- **ECS Agent** runs on each EC2 instance to communicate with the ECS control plane.  

### **2. Task Definition**  
- **Blueprint** for running containers (similar to `docker run` command).  
- Contains:  
  - **Container image** (from ECR/Docker Hub).  
  - **CPU/Memory allocation**.  
  - **Networking** (ports, network mode).  
  - **Volumes** (bind mounts, EFS, etc.).  
- Can define **one or more containers** (logically grouped).  

### **3. Task**  
- A **running instance** of a task definition.  
- If a task definition has **2 containers**, running **3 tasks** will launch **6 containers**.  
- Managed by **ECS scheduler** (decides where to place tasks).  

### **4. Service**  
- Ensures **long-running tasks** (like a web server).  
- Handles **scaling, load balancing, and self-healing**.  
- Works with **ALB/NLB** for traffic distribution.  

### **5. Elastic Container Registry (ECR)**  
- **AWS private Docker registry**.  
- Stores and manages **Docker images** securely.  

---  

## **Launch Types**  
### **1. EC2 Launch Type**  
- Manages **your own EC2 instances** (container-optimized AMIs).  
- You control **scaling, patching, and maintenance**.  
- Can SSH into instances and run Docker commands.  

### **2. Fargate (Serverless)**  
- No EC2 instances to manage.  
- AWS handles **scaling, patching, and resource allocation**.  
- Pay only for **resources used by tasks**.  

---  

## **Workflow**  
1. **Push Docker Image** → **ECR** (or Docker Hub).  
2. **Create Task Definition** (specify containers, networking, volumes).  
3. **Create ECS Cluster** (EC2 or Fargate).  
4. **Run Tasks/Services** (manually or via scheduler).  
5. **ECS Scheduler** places tasks on container instances.  
6. **Auto Scaling & Load Balancing** (if configured).  

---  

## **Key Takeaways**  
- ECS is AWS’s **native container orchestrator**.  
- Works with **Docker containers only**.  
- **Task Definition** = Blueprint, **Task** = Running instance.  
- **EC2 Launch Type** → You manage instances.  
- **Fargate** → Serverless (AWS manages infrastructure).  
- **ECR** = AWS private Docker registry.  

---  
**Next Steps:**  
- Demo: Creating an **ECS Cluster, Task Definition, and Service**.  
- Understanding **ECS Fargate** in depth.
- 

***################################################################################***

**32**

**################################################################################**
# **Amazon ECS Core Components - Summary Notes**

## **1. ECS Hierarchy Overview**
- **ECS Cluster**  
  - Top-level component.  
  - Contains one or more **container instances** (EC2 instances preloaded with Docker daemon & ECS agent).  

- **Service**  
  - Manages one or more **tasks**.  
  - Ensures tasks are running (auto-recovery, scaling).  
  - Can have multiple services in a cluster.  

- **Task**  
  - Created from a **task definition** (blueprint).  
  - Can run one or more **containers**.  
  - Tasks can span multiple container instances.  

- **Task Definition**  
  - Specifies:  
    - Docker images  
    - CPU/memory allocation  
    - Networking (ports, ENI)  
    - Volumes (storage)  
    - Environment variables  

- **Container**  
  - The lowest-level component.  
  - Runs the actual application from a Docker image.  

---

## **2. Amazon ECR (Elastic Container Registry)**
- **Private Docker Registry on AWS** (similar to Docker Hub but AWS-hosted).  
- **Features:**  
  - Secure, scalable, reliable.  
  - Supports private repositories.  
  - IAM-based access control.  
  - Uses standard Docker CLI commands (`push`, `pull`, `login`).  

### **ECR Components**
1. **Registry**  
   - Automatically provided per AWS account.  
   - Contains multiple **repositories**.  

2. **Authentication Token**  
   - Required for Docker client to authenticate with ECR.  
   - Obtained via AWS CLI (`aws ecr get-login-password`).  

3. **Repository**  
   - Stores Docker images (similar to Docker Hub repos).  

4. **Image**  
   - The actual Docker container image.  

---

## **3. Workflow for Using ECR**
1. **Install Prerequisites:**  
   - AWS CLI (for authentication).  
   - Docker (to build/push/pull images).  

2. **Build Docker Image Locally** (or on EC2).  

3. **Authenticate Docker with ECR:**  
   ```sh
   aws ecr get-login-password | docker login --username AWS --password-stdin <ECR-URI>
   ```  

4. **Push Image to ECR:**  
   ```sh
   docker tag <local-image> <ECR-URI>/<repo-name>:<tag>
   docker push <ECR-URI>/<repo-name>:<tag>
   ```  

5. **Use Image in ECS:**  
   - Reference ECR image URI in **ECS Task Definition**.  

---

## **Key Takeaways**
- **ECS Cluster** → **Service** → **Task** → **Container** (hierarchy).  
- **ECR** is AWS’s private Docker registry (alternative to Docker Hub).  
- **Task Definition** = Blueprint for running containers.  
- **Authentication** is required before pushing/pulling images from ECR.  




***################################################################################***

**33**

**################################################################################**
### **Notes: Creating an ECR Repository and Pushing/Pulling Docker Images**

#### **1. Introduction to AWS ECR**  
- **ECR (Elastic Container Registry)** is AWS's private Docker registry (similar to Docker Hub but hosted on AWS).  
- Used to store, manage, and deploy Docker container images.  

---

#### **2. Creating an ECR Repository**  
1. Go to the **AWS Console** → Search for **ECR**.  
2. Click **"Create repository"**.  
   - Provide a **repository name** (e.g., `add-emp`, `get-emp`).  
   - Click **"Create repository"**.  
3. After creation, note the **repository URI** (used for pushing/pulling images).  

---

#### **3. Pushing Docker Images to ECR**  
##### **Prerequisites:**  
- Docker installed on the local machine/EC2 instance.  
- AWS CLI configured with an IAM user having **ECR permissions**.  

##### **Steps:**  
1. **Configure AWS CLI** (if not already done):  
   ```bash
   aws configure
   ```
   - Enter **Access Key ID**, **Secret Access Key**, **Region**, and **Output format**.  

2. **Authenticate Docker with ECR:**  
   ```bash
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
   ```
   - This allows Docker to push/pull images from ECR.  

3. **Tag the Docker Image:**  
   ```bash
   docker tag <local-image-name>:<tag> <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>
   ```
   Example:  
   ```bash
   docker tag add-emp:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/add-emp:latest
   ```

4. **Push the Image to ECR:**  
   ```bash
   docker push <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>
   ```
   Example:  
   ```bash
   docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/add-emp:latest
   ```

5. **Verify in AWS Console:**  
   - Go to **ECR** → Select the repository → Check if the image appears.  

---

#### **4. Pulling Docker Images from ECR**  
1. **Authenticate Docker with ECR (if not logged in):**  
   ```bash
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
   ```

2. **Pull the Image:**  
   ```bash
   docker pull <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>
   ```
   Example:  
   ```bash
   docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/add-emp:latest
   ```

3. **Verify:**  
   ```bash
   docker images
   ```
   - Check if the pulled image appears in the list.  

---

#### **5. Additional AWS ECR Commands**  
- **List Repositories:**  
  ```bash
  aws ecr describe-repositories
  ```
- **List Images in a Repository:**  
  ```bash
  aws ecr describe-images --repository-name <repository-name>
  ```
- **Delete a Repository:**  
  ```bash
  aws ecr delete-repository --repository-name <repository-name> --force
  ```
  (Or delete via AWS Console.)  

---

#### **6. Summary**  
| **Action**          | **Command** |
|---------------------|-------------|
| **Login to ECR**    | `aws ecr get-login-password ...` |
| **Tag Image**       | `docker tag <image> <ecr-repo-uri>` |
| **Push Image**      | `docker push <ecr-repo-uri>` |
| **Pull Image**      | `docker pull <ecr-repo-uri>` |
| **List Repos**      | `aws ecr describe-repositories` |
| **Delete Repo**     | `aws ecr delete-repository` |

---

#### **7. Use Case**  
- Store Docker images in ECR for **ECS (Elastic Container Service)** deployments.  
- Private, secure, and scalable alternative to Docker Hub.  

---
### **Next Steps**  
- Deploy the pulled images in **ECS** or **Kubernetes (EKS)**.  
- Automate using **CI/CD pipelines (AWS CodePipeline, GitHub Actions)**.  



***################################################################################***

**34**

**################################################################################**
### **ECS Cluster Notes**  

#### **1. Introduction to ECS Cluster**  
- **Topmost component** in the ECS hierarchy.  
- A **logical grouping** of:  
  - Tasks / Services  
  - EC2 instances (if using EC2 launch type)  
- **Region-specific** → Must be created in a specific AWS region.  

---

#### **2. Cluster Compatibility Options**  
When creating a cluster, you must choose between:  

##### **A. EC2 + Networking**  
- **EC2 instances** run Docker containers.  
- Two sub-options:  
  1. **EC2 Linux + Networking** → For Linux containers.  
  2. **EC2 Windows + Networking** → For Windows containers.  
- **Key Points:**  
  - Instances run **Docker daemon + ECS agent**.  
  - Launched within a **VPC (private network)**.  
  - Requires manual scaling & management of EC2 instances.  

##### **B. Networking Only (Fargate Launch Type)**  
- **Serverless** → No EC2 instances to manage.  
- AWS handles:  
  - Provisioning  
  - Scaling  
  - Infrastructure management  
- Simply define **task definitions**, and AWS runs them.  

---

#### **3. Key Concepts**  

##### **ECS Container Instance (EC2 Launch Type)**  
- An **EC2 instance** registered with ECS.  
- Must have:  
  - **ECS Agent** (communicates with ECS service).  
  - **Docker daemon** (runs containers).  
- Requires an **IAM role** (`AmazonEC2ContainerServiceforEC2Role`) to interact with ECS.  
  - Allows actions like:  
    - Creating clusters  
    - Polling for tasks  
    - Fetching authorization tokens  

##### **Networking in ECS**  
- **EC2 Launch Type:**  
  - Runs inside a **VPC** (must be configured beforehand).  
- **Fargate Launch Type:**  
  - Also uses VPC but abstracts networking details.  

---

#### **4. Hands-On: Creating an ECS Cluster**  
- **Steps:**  
  1. Choose **cluster compatibility** (EC2 or Fargate).  
  2. Configure **networking (VPC, subnets, security groups)**.  
  3. For **EC2 launch type**, ensure:  
     - IAM role (`AmazonEC2ContainerServiceforEC2Role`) is attached.  
     - Docker and ECS agent are running.  
  4. For **Fargate**, just define tasks—AWS handles the rest.  

---

### **Summary Table**  
| Feature | EC2 Launch Type | Fargate Launch Type |  
|---------|----------------|---------------------|  
| **Management** | Manual (EC2 instances) | Serverless (AWS-managed) |  
| **Scaling** | Manual/Auto Scaling | Automatic |  
| **Networking** | Uses VPC | Uses VPC (simplified) |  
| **Use Case** | Full control over infrastructure | No infrastructure management |  

---

### **Next Steps**  
- **Demo:** Deploying an application using **both EC2 & Fargate launch types**.  
- **Explore:** Task definitions, services, and load balancing in ECS.  


***################################################################################***

**35 LAb Ecs cluster**

**################################################################################**
### **Amazon ECS Cluster Creation Notes**  

#### **1. ECS Console Overview**  
- **ECS Clusters**: Where clusters are created and managed.  
- **Task Definitions**: Define how containers should run (covered in the next video).  
- **ECR Repositories**: Stores Docker images (already created: `add-mp`, `get-mp`, `ecs`).  
  - `add-mp` has an image tagged `latest`.  
  - `get-mp` has an image tagged `get-mp:latest`.  
- **Amazon EKS**: Managed Kubernetes service (not covered in detail here).  

---  

#### **2. Creating an ECS Cluster**  
**Cluster Templates Available:**  
1. **Networking Only** (Fargate-powered, most convenient).  
2. **EC2 Linux + Networking** (Manual EC2 provisioning).  
3. **Windows + Networking** (For Windows containers).  

**Selected:** **EC2 Linux + Networking** (for this demo).  

---  

#### **3. Cluster Configuration**  
- **Cluster Name**: Assign a name (or create empty).  
- **Provisioning Model**:  
  - **On-Demand Instances** (fixed hourly pricing).  
  - **Spot Instances** (cheaper, but can terminate anytime).  
- **Instance Type**:  
  - **T2.micro** (Free Tier eligible).  
  - Other options:  
    - **Compute Optimized** (C3, C4, C5).  
    - **Memory Optimized** (R3, R4).  
    - **Storage Optimized** (D2, I2, I3).  
    - **General Purpose** (M3, M4, M5, T2, T3).  
    - **GPU Instances** (P2).  
- **Number of Instances**: Set to **3** (for demo).  
- **AMI ID**: Pre-configured with Docker and ECS agent.  
- **Storage**: Minimum **22GB EBS** (sufficient for demo).  
- **Key Pair**: Required for SSH access to EC2 instances.  

---  

#### **4. Networking Setup**  
- **VPC**: Use existing AWS VPC.  
- **Subnets**:  
  - Selected **private subnets** (avoid public subnet for now).  
- **Security Group**: Attach existing private subnet security group.  
- **IAM Role**:  
  - Required for ECS agent to make API calls.  
  - Created a new role during setup.  
- **Tags**: Added for identification.  

---  

#### **5. Cluster Creation via CloudFormation**  
- ECS uses **CloudFormation** to deploy the cluster.  
- **Steps in CloudFormation Stack:**  
  1. **Launch Configuration** created.  
  2. **Auto Scaling Group** created.  
  3. **ECS Cluster** deployed.  
- **Viewing Stack Details**:  
  - Can check YAML/JSON template.  
  - Shows instance count, VPC, subnets, etc.  

---  

#### **6. Post-Creation Checks**  
- **Cluster Status**: **Active**.  
- **No Running Tasks/Services** (since no task definition yet).  
- **EC2 Instances**:  
  - 3 instances launched (2 in one AZ, 1 in another).  
  - Private IPs assigned from VPC range.  
- **Auto Scaling Group**:  
  - **Min: 0**, **Max: 3**, **Desired: 3**.  
  - Can modify scaling policies later.  

---  

#### **7. Next Steps**  
- **Create Task Definition** (to define container specs).  
- **Launch Services** (to run tasks).  
- **Scaling Policies** (to adjust EC2 instances dynamically).  

---  

### **Key Takeaways**  
✅ ECS clusters can be **EC2-backed** or **Fargate-powered**.  
✅ **CloudFormation automates** infrastructure deployment.  
✅ **Auto Scaling Group** manages EC2 instances.  
✅ **Task Definitions & Services** are needed to run containers.  

Next: **Task Definitions** → How containers are defined and deployed. 🚀

***################################################################################***

**36 Task Definition concepts**

**################################################################################**
### **Notes on Task Definition in AWS ECS**  

#### **1. What is a Task Definition?**  
- Required to run Docker containers in **ECS** (Elastic Container Service).  
- Similar to `docker run` command but defined as a JSON configuration.  
- Specifies:  
  - **Docker image** (from Amazon ECR or Docker Hub).  
  - **CPU & Memory** allocation.  
  - **Launch type** (EC2 or Fargate).  
  - **Networking mode** (backed by VPC).  
  - **Logging configuration**.  
  - **Commands** to run inside the container.  
  - **Data volumes** (if needed).  
  - **IAM roles** (different from the cluster IAM role).  

#### **2. Key Components of a Task Definition**  
| Component | Backed By | Notes |
|-----------|----------|-------|
| **Docker Image** | Amazon ECR | Image must be pushed to ECR first. |
| **CPU & Memory** | EC2 Instance Type | If using EC2 launch type, depends on the instance (e.g., t2.micro has limited resources). |
| **Networking** | VPC | Must be configured when setting up the cluster. |
| **Volumes** | EBS | EC2 instances in the cluster have attached EBS storage (default: 22GB). |

#### **3. Multi-Container Task Definitions**  
- A single task definition can include **multiple containers**.  
- Example:  
  - If a task definition has **2 containers** and you run **2 tasks**, then **4 containers** will be running.  
- **When to use multiple containers in one task definition?**  
  - If containers **share a common purpose** and **scale together**.  
  - Example: A frontend and backend that must always run together.  

#### **4. Best Practices for Task Definitions**  
- **Separate Containers into Different Task Definitions if:**  
  - They have **different scaling needs** (e.g., one is used more frequently than the other).  
  - Example:  
    - **Get Employee API** (high traffic) → Separate task definition.  
    - **Update Employee API** (low traffic) → Separate task definition.  
- **Group Related Containers in the Same Task Definition if:**  
  - They **work together** (e.g., frontend + backend).  

#### **5. Task Definition vs. Service**  
- **Task Definition** = Blueprint (defines how containers should run).  
- **Service** = Runs and maintains instances of task definitions (ensures desired count is met).  

#### **6. Example Use Case (3-Tier App)**  
| Service | Task Definition Strategy | Reasoning |
|---------|--------------------------|-----------|
| **Frontend (AngularJS)** | Separate Task Definition | Scales independently from backend. |
| **Backend (Python/Java API)** | Separate Task Definition | Different resource needs. |
| **Database (Data Store)** | Separate Task Definition | Persistent storage, different scaling. |

### **Key Takeaways**  
✔ **Task definitions** define how containers run in ECS.  
✔ **Group containers logically** (same scaling needs → same task definition).  
✔ **Use services** to run and maintain task instances.  
✔ **EC2 launch type** depends on underlying instance resources.  
✔ **Fargate** abstracts infrastructure but still requires CPU/memory specs.  


***################################################################################***

**37 Task definition 1 component n/w mode**

**################################################################################**
Here are the key notes on creating an **ECS Task Definition** and its components:

---

### **1. Launch Type Compatibility**
- **Options**: Fargate or EC2  
  - **Fargate**: Only supports **awsvpc** network mode.  
  - **EC2**: Supports multiple network modes (bridge, host, awsvpc, none).  

---

### **2. Task Definition Components**
#### **A. Family Name**
- Acts as the task definition name.  
- Revisions are numbered sequentially (e.g., `my-task:1`, `my-task:2` for updates).  

#### **B. Task Role**
- IAM role granting permissions to containers to interact with AWS services (e.g., RDS, S3).  
- Example: Allows an app container to access an RDS MySQL database.  

#### **C. Task Execution Role**
- Permissions for ECS to:  
  - Pull images from **ECR**.  
  - Publish logs to **CloudWatch**.  

#### **D. Network Mode**
- **Bridge**: Default Docker virtual network. Requires manual host-port mapping.  
- **Host**: Bypasses Docker networking; container ports map directly to EC2 instance ports.  
- **awsvpc**: Best for high performance; uses EC2’s VPC networking with **dynamic port mapping**.  
- **None**: No external connectivity.  

#### **E. Container Definitions**
- Equivalent to `docker run` parameters (image, ports, environment variables, etc.).  
- A task can include **multiple containers** (e.g., app + sidecar containers).  

#### **F. Volumes**
- **Types**:  
  - **Docker volumes** (managed by Docker).  
  - **Bind mounts** (map host directories to containers).  
- Shared across containers in the task.  

#### **G. Task Placement Constraints**
- Custom rules for placing tasks on EC2 instances (rarely modified).  

#### **H. Task Size**
- Total CPU/memory allocated to the task (shared across all containers in the task).  

---

### **3. Networking Modes Deep Dive**
#### **Bridge Mode**
- Requires **host-port:container-port** mapping (like `-p 8080:80` in Docker).  
- Risk of port conflicts if multiple tasks use the same host port.  

#### **Host Mode**
- Container port = Host port (e.g., container port `80` → host port `80`).  
- No flexibility; port conflicts can occur.  

#### **awsvpc Mode**
- **Dynamic port mapping**: Host port is auto-assigned by ECS.  
- Best for scalability (no manual port management).  

---

### **4. Creating a Task Definition (EC2 Example)**
1. **Name**: Enter a family name (e.g., `my-task-def`).  
2. **Network Mode**: Select `awsvpc` (recommended) or `bridge`/`host`.  
3. **Container Definitions**:  
   - **Image**: Specify ECR/registry path.  
   - **Port Mappings**:  
     - `bridge`: Define both host and container ports.  
     - `host`/`awsvpc`: Only container port (host port handled automatically).  
4. **Volumes (Optional)**: Attach Docker/bind mounts.  
5. **Task Role/Execution Role**: Attach IAM roles if needed.  

---

### **Key Takeaways**
- **awsvpc** simplifies networking with dynamic ports.  
- **Task Role** enables AWS service access (e.g., RDS).  
- **Execution Role** is critical for ECR/CloudWatch access.  
- Use **volumes** for persistent or shared storage.  

***################################################################################***

**38 Container definitin**

**################################################################################**
### **Docker Container Run Command Options**  

#### **Basic Options**  
1. **`-a` or `--attach`**  
   - Attach to a running container.  

2. **`-d` or `--detach`**  
   - Run the container in the background (widely used).  

3. **`-e` or `--env`**  
   - Pass environment variables (e.g., database credentials).  

4. **`--name`**  
   - Assign a name to the container.  

5. **`--net`**  
   - Specify the network for the container (allows inter-container communication using container names).  

6. **`-p` or `--publish`**  
   - Map a host port to a container port (used with `EXPOSE` in Dockerfile).  

7. **`-v` or `--volume`**  
   - Used for Docker volumes and bind mounts.  

---

#### **Advanced Options**  
1. **`--cpus`**  
   - Limit the number of CPUs a container can use.  

2. **`--dns`**  
   - Set custom DNS servers.  

3. **`--entrypoint`**  
   - Override the default `ENTRYPOINT` from the Dockerfile.  

4. **`--health-cmd`**  
   - Define a command to check container health (e.g., `curl` to test a web server).  

5. **`--health-interval`**  
   - Time between health checks.  

6. **`--health-retries`**  
   - Number of consecutive failures before marking a container as unhealthy.  

7. **`--hostname`**  
   - Set the container hostname.  

8. **`--link`**  
   - Link containers for inter-container communication (legacy method).  

9. **`--log-driver`**  
   - Specify the logging driver (e.g., `json-file`, `syslog`, `fluentd`).  

10. **`--memory` (`-m`)**  
    - Hard memory limit (container is killed if exceeded).  

11. **`--memory-reservation`**  
    - Soft memory limit (container can exceed but is throttled).  

12. **`--workdir`**  
    - Override the working directory (`WORKDIR` in Dockerfile).  

13. **`--ulimit`**  
    - Set Linux system resource limits (e.g., file descriptors, processes).  

14. **`--volumes-from`**  
    - Mount volumes from another container.  

15. **`--ip`**  
    - Assign a specific IP address to the container.  

16. **`--mount`**  
    - Attach a filesystem mount to the container.  

---

### **Mapping Docker Run Options to ECS Task Definition**  
| **Docker Run Option**       | **ECS Task Definition Equivalent** |
|-----------------------------|-----------------------------------|
| `--memory` / `--memory-reservation` | Memory Limits (Hard/Soft) |
| `-p` / `--publish`          | Port Mappings |
| `--health-cmd`, `--health-interval`, `--health-retries` | Health Check (Command, Interval, Timeout, Retries) |
| `-e` / `--env`              | Environment Variables |
| `--cpus`                    | CPU Units (vCPU) |
| `--entrypoint`              | Override Entrypoint |
| `--workdir`                 | Working Directory |
| `--link`                    | Links (Deprecated in favor of service discovery) |
| `--hostname`                | Hostname |
| `--dns`                     | DNS Servers |
| `--log-driver`              | Log Configuration (CloudWatch, Fluentd, Splunk, etc.) |
| `--ulimit`                  | Resource Limits (File size, CPU, etc.) |
| `--volumes-from`            | Volumes From (Advanced) |
| `--mount`                   | Mount Points (Storage) |

---

### **Key Takeaways**  
- Most `docker run` options have direct equivalents in ECS Task Definitions.  
- Some options (like `--link`) are legacy and replaced by service discovery.  
- ECS integrates with AWS services like **CloudWatch Logs** for logging.  
- **Health checks** are crucial for monitoring container status.  
- **Memory and CPU limits** prevent resource exhaustion.  

This helps in understanding how Docker container configurations translate to AWS ECS Task Definitions. 🚀
***################################################################################***

**39 Task efinition coponent volumes**

**################################################################################**
### **Notes on ECS Volumes in Task Definitions**

#### **1. Overview of Volumes in ECS Task Definitions**
- When creating a **task definition**, you can optionally specify **volumes**.
- These volumes are passed to **container instances** and can be accessed by:
  - The same container.
  - Other containers in the same task.

---

#### **2. Types of Data Volumes**
There are **two types** of data volumes in ECS:

##### **A. Docker Volumes**
- **Managed by Docker** and stored under `/var/lib/docker/volumes` on the **EC2 instance**.
- **Volume Drivers** can integrate with external storage (e.g., **EBS**).
  - Default: **Local volume driver** (most common).
  - Third-party drivers (rarely used).
- **Supported only with EC2 launch type** (not Fargate).

##### **B. Bind Mounts**
- A **file/directory on the host machine** is **mapped** to the container’s file system.
- **Supported for both EC2 & Fargate launch types**.

---

#### **3. Docker Volumes in Practice**
- Defined in the **Dockerfile** using the `VOLUME` directive:
  ```dockerfile
  VOLUME /var/www/html
  ```
- When the container runs **without `-v` flag**, Docker **automatically**:
  - Creates a volume in `/var/lib/docker/volumes/` on the host.
  - Maps it to the container’s specified path (`/var/www/html`).
- **Named volumes** (not covered here) allow explicit volume naming.

---

#### **4. Bind Mounts in Practice**
- Requires **explicit mapping** using `-v` or `--volume` in `docker run`:
  ```bash
  docker run -v /home/user/container-volume:/var/www/html ...
  ```
  - **Host path (`/home/user/container-volume`)** → **Container path (`/var/www/html`)**.
- **Task Definition Setup**:
  1. **Add a volume** in the task definition:
     - **Name**: e.g., `x`
     - **Source Path**: Host path (e.g., `/home/ec2-user/container-volume`)
  2. In **container definition**, specify **mount points**:
     - **Container Path**: e.g., `/var/www/html`
     - **Source Volume**: `x` (matches the volume name in task definition).

---

#### **5. Key Differences**
| Feature          | Docker Volumes | Bind Mounts |
|------------------|---------------|-------------|
| **Managed By**   | Docker        | User        |
| **Host Path**    | Auto-created (`/var/lib/docker/volumes/`) | User-specified (e.g., `/home/user/data`) |
| **ECS Support**  | **EC2 only**  | **EC2 & Fargate** |

---

#### **6. Practical Use Cases**
- **Docker Volumes**: Best for **persistent storage managed by Docker** (e.g., databases, logs).
- **Bind Mounts**: Best for **direct host-container file sharing** (e.g., config files, static web content).

---

#### **7. Task Definition Configuration**
- **For Docker Volumes**:
  - No need to add a volume in task definition (handled by Docker).
  - Just specify **mount points** in container definition.
- **For Bind Mounts**:
  1. **Add a volume** in task definition (host path).
  2. **Map it** in container mount points.

---

### **Summary**
- **Docker Volumes** → Managed by Docker, EC2-only.
- **Bind Mounts** → Manual host-container mapping, works with **EC2 & Fargate**.
- **Task Definition Setup** differs slightly for each type.

🚀 **Next**: Explore hands-on labs for Docker volumes & bind mounts in **Section 2.6**.



***################################################################################***

**40 Create a complete Task Definition**

**################################################################################**
Here are the key notes from the provided content on creating an AWS ECS Task Definition:

### **Components Needed Before Creating Task Definition**
1. **AWS ECR Repository**
   - Already created with two images:
     - `AddMP:latest`
     - `GetMP:latest`

2. **VPC Setup**
   - Contains:
     - 2 private subnets
     - 1 public subnet

3. **Database (RDS)**
   - **Instance Name:** `employee-db-instance`
   - **Engine:** MySQL
   - **Region:** `us-east-1b`
   - **Instance Type:** `db.t2.micro`
   - **Deployed in private subnets**
   - **Endpoint & Port:** Available at port `3306`
   - **Credentials:**
     - Username: `root`
     - Password: `Abcd1234`
   - **Database Name:** `employee`

4. **IAM Role for ECS Task**
   - **Role Name:** `employee-application-task-role`
   - **Permissions:** Full access (for demo, restrict in production)
   - **Trust Relationship:** Allows ECS tasks to assume this role.

---

### **Creating the Task Definition**
1. **Task Definition Type:** `EC2` (not Fargate)
2. **Basic Configuration:**
   - **Name:** `employee-application-task-dev`
   - **Task Role:** `employee-application-task-role`
   - **Network Mode:** `awsvpc` (VPC mode)
   - **Task Execution Role:** Auto-created by ECS
   - **Task Size:**
     - Memory: `256 MB`
     - CPU: `2 vCPU`

3. **Container Definitions (Multi-Container Task)**
   - **First Container:** `AddMP`
     - **Image:** ECR URI for `AddMP:latest`
     - **Memory Limits:**
       - Hard: `100 MB`
     - **Port Mappings:** `80` (exposed by container)
     - **Environment Variables:**
       - `DB_HOST`: RDS endpoint
       - `DB_PORT`: `3306`
       - `DB_USER`: `root`
       - `DB_PWD`: `Abcd1234`
       - `DATABASE`: `employee`
   - **Second Container:** `GetMP`
     - **Image:** ECR URI for `GetMP:latest`
     - **Memory Limits:**
       - Hard: `128 MB`
       - Soft: `64 MB`
     - **Port Mappings:** `8080` (exposed by container)
     - **Environment Variables:** Same as `AddMP`.

4. **Volumes:** Not required (apps interact directly with RDS).

5. **Tags:**
   - `employee-application`

6. **Task Definition Created:**
   - **Family Name:** `employee-application-task-dev`
   - **Revision ID:** `1` (increments on updates).

---

### **Post-Creation Actions**
1. **Run Task Directly**
   - Can launch tasks directly from the definition.
   - If `number of tasks = 2`, each runs 2 containers → Total **4 containers**.

2. **Create/Update Service**
   - Used for long-running tasks (e.g., web servers).
   - Manages scaling, load balancing, and availability.

3. **JSON View**
   - Task definition can be viewed/modified in JSON format.
   - Includes:
     - Container definitions (images, ports, env vars).
     - Network mode (`awsvpc`).
     - Memory/CPU allocations.
     - IAM role references.

---

### **Key Takeaways**
- **Task Definitions** are blueprints for running containers in ECS.
- **Multi-container tasks** allow co-located services (e.g., `AddMP` + `GetMP`).
- **VPC Mode** ensures containers use private subnets for security.
- **Environment Variables** securely pass DB credentials (avoid hardcoding).
- **IAM Roles** grant permissions to ECS tasks (least privilege in production).

Next Steps:  
- Create an **ECS Cluster** and **Service** to deploy the task definition.  
- Use **Load Balancers** if exposing services publicly.  


***################################################################################***

**41 Bridge networking mode TDEF component**

**################################################################################**
### **ECS Task Definition Components - Networking Mode (Bridge)**

#### **Prerequisites:**
1. **Cluster Creation:**
   - Created a cluster named **`components-demo-cluster`**.
   - Status: **Active**.
   - Contains **1 running container instance** (EC2 - `t3.micro`).
   - No running tasks or services.

2. **ECR Repository:**
   - Created a repository named **`dev-components`**.
   - No existing images; needs to push a Docker image.

---

### **Steps to Push Docker Image to ECR:**
1. **Dockerfile Used:**
   ```dockerfile
   FROM centos:latest
   RUN yum install httpd -y
   COPY index.html /var/www/html/
   COPY image.png /var/www/html/
   EXPOSE 80
   ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
   ```
   - Simple Apache web server running on **port 80**.

2. **Build and Test Locally:**
   - Built image: `docker build -t dev:latest .`
   - Ran container: `docker run -d -p 8080:80 dev:latest`
   - Tested locally: `http://localhost:8080` (working).

3. **Push to ECR:**
   - Authenticated Docker to ECR:
     ```bash
     aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
     ```
   - Tagged image:
     ```bash
     docker tag dev:latest <account-id>.dkr.ecr.<region>.amazonaws.com/dev-components:latest
     ```
   - Pushed image:
     ```bash
     docker push <account-id>.dkr.ecr.<region>.amazonaws.com/dev-components:latest
     ```
   - Verified in ECR console.

---

### **Task Definition Setup (Bridge Networking Mode)**
1. **Task Definition Details:**
   - **Name:** `tdf-components-demo`
   - **Launch Type:** EC2
   - **Network Mode:** `bridge`
   - **Task Size:**
     - CPU: **1 vCPU** (since `t3.micro` has 2 vCPUs, max 2 tasks per instance).
     - Memory: **64 MB** (low to fit in `t3.micro`'s 1GB RAM).

2. **Container Definition:**
   - **Name:** `apache-container`
   - **Image URI:** ECR image URI (`dev-components:latest`)
   - **Port Mappings:**
     - **Container Port:** `80`
     - **Host Port:** `8080` (explicitly mapped).

3. **No Volumes/Advanced Configurations:**
   - Skipped health checks, environment variables, etc.

4. **Created Task Definition.**

---

### **Running the Task**
1. **Run Task in Cluster:**
   - Cluster: `components-demo-cluster`
   - Launch Type: EC2
   - Number of Tasks: `1`
   - Task Group: `bridge`
   - **No VPC/Security Group** (since using `bridge` mode).

2. **Task Execution:**
   - Task started successfully on the single EC2 instance.
   - Verified in EC2 instance:
     ```bash
     docker ps
     ```
     - Shows container running with `8080:80` mapping.
   - Accessed app via EC2 public IP: `http://<EC2-IP>:8080`.

3. **Attempt to Run Second Task:**
   - Failed with error: **"Resource:PORTS"**.
   - Reason: Port `8080` already in use on the host (bridge mode requires unique host ports).

4. **Scaled Cluster:**
   - Added **second EC2 instance** (scaled to 2 instances).
   - Ran task again → Success (task ran on the new instance).

---

### **Key Observations:**
1. **Bridge Networking Mode:**
   - Explicit host port mapping (`hostPort:containerPort`).
   - **Cannot run multiple tasks on the same instance** if they use the same host port.
   - Requires **separate EC2 instances** for multiple tasks (inefficient for scaling).

2. **Dynamic Port Mapping Limitation:**
   - Unlike **`awsvpc`** mode (which allows dynamic ports), `bridge` mode requires manual port management.

3. **Use Case:**
   - Suitable for **legacy applications** relying on Docker’s default bridge network.
   - Not ideal for scalable microservices.

---

### **Next Steps:**
- **Host Networking Mode:**  
  - Containers use host’s network directly (no NAT).
  - Still faces port conflicts but with different behavior.
- **awsvpc Mode:**  
  - Assigns ENI to each task (best for scalability + security).

---
### **Summary of Actions:**
1. Created cluster + ECR repo.
2. Built/pushed Docker image.
3. Created task definition (`bridge` mode).
4. Ran task → Success (1 instance).
5. Failed on second task (port conflict).
6. Scaled cluster → Success (2 instances).

**Conclusion:**  
`bridge` mode is restrictive for multi-task deployments. **`awsvpc`** (covered later) solves this.  



***################################################################################***

**42 Host networking mode TDEF component**

**################################################################################**
# **Lab Notes: Host Networking Mode and VPC Networking Mode in AWS ECS**

## **1. Host Networking Mode**
### **Prerequisites:**
- **ECS Cluster**: Running with **2 components**, but only **1 EC2 instance** is active.
  - **Instance ID**: Visible in the ECS console.
  - **Status**: Active.
  - **Registered Container Instances**: 1.
- **ECR Repository**: Contains the Docker image (`components:latest`) used in previous demos.
- **Task Definition**: Already created with **Revision ID 1** (`components-demo:1`).

### **Modifying Task Definition for Host Networking**
- To change networking mode, create a **new revision** (similar to versioning).
- **Steps**:
  1. Open the existing task definition.
  2. Click **Create new revision**.
  3. Change **Networking Mode** from `bridge` to `host`.
  4. **No host port mapping needed** (automatically binds container port `80` to host port `80`).
  5. **New Revision ID**: `4` (earlier revisions were created for testing).

### **Running a Task in Host Mode**
- **Launch Type**: `EC2` (since host mode requires EC2 instances).
- **Task Group Name**: `host`.
- **Number of Tasks**: `1`.
- **Result**: Task runs successfully on the existing EC2 instance.

### **Verification**
- **SSH into the EC2 instance** and check running containers:
  ```bash
  docker ps
  ```
  - Shows the container running on **port 80** (mapped to host port `80`).
- **Access the application**:  
  - Open browser/curl to instance IP on port `80` → Application is accessible.

### **Limitation: Port Conflict**
- **Attempt to run a second task**:
  - Fails with error: **"Resource ports are already in use"**.
  - Reason: Only **one container per host port** (port `80` already occupied).
- **Solution**:  
  - **Scale the cluster** (add another EC2 instance).
  - **Run task again** → Now it launches on the **new instance**.

### **Key Observations**
- **Performance**: Host mode has **better networking performance** than bridge mode.
- **Limitation**:  
  - Only **one container per image** can run on an instance (due to port binding).
  - Similar issue exists in **bridge mode** (explicit host port mapping required).

---

## **2. VPC Networking Mode (Preview)**
*(Not fully detailed in the transcript but implied as the next topic.)*

### **Expected Differences from Host Mode**
- **AWS-managed networking** (no direct host port binding).
- **Elastic Network Interface (ENI) per task** → No port conflicts.
- **Supports multiple tasks per instance** (unlike host mode).
- **Better isolation & security** (each task gets its own IP).

### **Comparison Table**
| Feature          | Host Mode                     | Bridge Mode                   | VPC Mode                     |
|------------------|------------------------------|-------------------------------|------------------------------|
| **Performance**  | High (direct host networking) | Moderate (NAT via Docker)     | High (AWS-managed ENI)       |
| **Port Mapping** | Auto (container=host port)    | Manual (specify host port)    | No host port conflicts       |
| **Scalability**  | Limited (1 task per port)     | Limited (manual port mapping) | High (no port conflicts)     |
| **Use Case**     | Low-latency apps              | Legacy Docker compatibility   | Modern AWS-native workloads  |

---

## **Conclusion**
- **Host mode** improves performance but **restricts task density** per instance.
- **VPC mode** (next topic) resolves these issues with **better scalability** and **no port conflicts**.
- **Bridge mode** is legacy-compatible but **less efficient** than host/VPC modes.  



***################################################################################***

**43 AWS VPC networking mode**

**################################################################################**
### **Notes: VPC Network Component Demo in AWS ECS**

#### **Prerequisites**
- A cluster named **"components VPC"** is already created.
- No services or tasks are currently running on this cluster.
- An EC2 instance is running with instance type **t3.small** (chosen for sufficient ENIs).

---

### **Steps in the Demo**

#### **1. Task Definition Setup**
- **Repository**: Using the same repository (unchanged).
- **Task Definition**: 
  - Navigate to the existing task definition.
  - **Create a new revision** (last revision ID is used).
  - **Network Mode**: Changed to **"awsvpc"** (AWS VPC mode).
    - Containers in the task share an **Elastic Network Interface (ENI)**.
    - Port mappings only specify **container ports** (no host ports).
    - **Important**: If running multiple containers on the same host, multiple ENIs are needed.

#### **2. Running the Task**
- **Cluster**: Selected **"components VPC"**.
- **Subnets**: Chose **1C and 1D**.
- **Security Group**: Configured to allow SSH access.
- **Public IP**: Disabled (not needed for this demo).
- **Task Execution**:
  - Task status changes to **"running"**.
  - **No ports visible** in the container (due to dynamic port mapping in `awsvpc` mode).
  - **Cannot directly access** the app (requires an **Elastic Load Balancer** for reverse proxy).

#### **3. Running a Second Task**
- **Same Task Definition**: Launched another task in the same cluster.
- **Same EC2 Instance**: Both tasks run on the **same container instance** (only 1 EC2 instance registered).
- **Verification**:
  - Checked running containers inside the EC2 instance (`docker ps`).
  - **Two containers** are running with the same image.

#### **4. Key Observations**
- **AWS VPC Mode** allows **multiple containers** (same image) on a **single EC2 instance**.
- **ENI Limitation**:
  - **t3.small** was chosen because it supports multiple ENIs.
  - **t2.micro / t3.micro** would fail (insufficient ENIs).
  - Error: `Unable to assign Elastic Network Interface` if ENIs are insufficient.

#### **5. Comparison with Other Network Modes**
- **Bridge Mode**: Requires explicit host port mapping (port conflicts possible).
- **Host Mode**: Binds directly to host ports (only one container per port).
- **AWS VPC Mode**:
  - No port conflicts.
  - Requires **ELB** for access (dynamic ports).
  - Better isolation and scalability.

---

### **Next Steps**
- Use an **Elastic Load Balancer (Classic/ALB)** to expose the application.
- Discuss **ECS Services** in the next section for managing long-running tasks.

---

### **Key Takeaways**
✅ **AWS VPC mode** enables **multi-container deployments** on a single host.  
✅ **Dynamic port mapping** requires an **ELB** for accessibility.  
✅ **Instance type matters** (ensure enough ENIs for multiple tasks).  
✅ **Better than Bridge/Host mode** for avoiding port conflicts.  

---
**Demo by: [Instructor's Name]** | **AWS ECS - VPC Networking Mode**  
**Next Topic: ECS Services & Load Balancing**


***################################################################################***

**44 Volumes TDEF component**

**################################################################################**
### **Notes: Task Definition - Volumes in AWS ECS**  

#### **Key Points Covered in the Video:**  
1. **Volumes in Task Definitions**  
   - Last component of task definition.  
   - Comparison between **bind mounts** and **Docker volumes**.  

2. **Prerequisites**  
   - Existing AWS ECS cluster (reusing previous setup).  
   - New Docker image with a **volume directive** in the Dockerfile.  

3. **Dockerfile Modification**  
   - Added `VOLUME ["/var/www/html"]` to specify a mount point in the container.  
   - Built and tagged the image as `x-volume`.  
   - Pushed to a new Docker repository.  

4. **Task Definition Setup (Docker Volumes)**  
   - Created a new task definition (EC2 launch type, host network mode).  
   - Added container details:  
     - Image: `x-volume`  
     - Port mapping: `80:80`  
     - **No explicit volume mount** (default Docker volume behavior).  
   - Ran the task and verified:  
     - Docker automatically creates a volume (`/var/lib/docker/volumes/...`).  
     - Changes in the volume reflect in the container (e.g., modified `index.html`).  

5. **Bind Mounts Demonstration**  
   - Created a **new task revision** with a **bind mount**:  
     - Host path: `/home/ec2-user/dev-volumes`  
     - Container path: `/var/www/html`  
   - Key differences from Docker volumes:  
     - **Bind mounts work "backwards"**—host directory overrides container directory.  
     - If the host directory is empty, the container directory appears empty.  
     - Changes must be made on the **host first**, then reflected in the container.  
   - Tested by creating `index.html` on the host → visible in the container.  

6. **Comparison Summary**  
   | **Feature**       | **Docker Volumes**                          | **Bind Mounts**                          |  
   |-------------------|--------------------------------------------|------------------------------------------|  
   | **Managed by**    | Docker                                     | User-specified host path                 |  
   | **Persistence**   | Survives container removal                 | Tied to host directory                   |  
   | **Initial Data**  | Retains container directory contents       | Overrides container directory            |  
   | **Use Case**      | Persistent data (DB, logs)                 | Host-container file sharing (configs)    |  

7. **Next Topic: Services**  
   - **Run Task** is for testing; **Services** are for production.  
   - Services provide:  
     - Load balancing integration.  
     - Auto-scaling capabilities.  
     - Task lifecycle management (restart on failure).  

---  
### **Key Takeaways**  
- **Docker volumes** are managed by Docker and persist independently of containers.  
- **Bind mounts** link host directories directly to containers (host files take precedence).  
- **Task definitions** must explicitly define volumes for bind mounts.  
- **Services** (next topic) abstract task management for production workloads.  

Would you like a simplified step-by-step guide for implementing volumes in ECS? 😊

***################################################################################***

**45 TDEF summary**

**################################################################################**
### **Summary: Task Definition in AWS ECS**  

#### **1. Task Definition**  
- A **blueprint** for running containers in ECS.  
- Contains **one or more container definitions** (similar to `docker run` options).  
- Specifies:  
  - Docker image, CPU/memory (**task size**), networking (**network mode**), volumes, environment variables, etc.  

#### **2. ECS Cluster**  
- A **group of EC2 instances** (managed via Auto Scaling).  
- Tasks run on these instances.  

#### **3. Running a Task**  
- A **task** is an **instance of a task definition** (like a running containerized application).  
- Example:  
  - **Task Definition** = Blue Container + Green Container.  
  - **Run Task 1** → Launches **Blue + Green Containers (Instance 1)**.  
  - **Run Task 2** → Launches **Blue + Green Containers (Instance 2)**.  

#### **4. Next Step: ECS Services**  
- Instead of manually running tasks, **ECS Services** help maintain desired task count (auto-recovery, scaling).  

### **Key Takeaway**  
- **Task Definition** = Container configuration (like `docker run`).  
- **Task** = Running instance of the definition.  
- **Cluster** = Group of EC2 instances where tasks run.  
- **Services** (next topic) = Managed way to run tasks.  

Let me know if you'd like any refinements! 🚀


***################################################################################***

**46 Cases tudt TDEF & Run**

**################################################################################**
### **Notes on Implementing an Application Using ECS (Elastic Container Service)**  

#### **1. Overview**  
- Discussed **talkativeness definition** (likely a placeholder term).  
- Proceeding to implement an application using **ECS** with **class definitions** (instead of a case study).  
- Need to **create a cluster** before deploying the application.  

---

#### **2. Cluster Creation**  
- **Cluster setup** is assumed to be known (skipped detailed steps).  
- Creating a **Limitlessness application cluster** (likely a typo, should be **ECS cluster**).  
- Cluster creation provisions an **EC2 instance** (needed for IP address configuration).  

---

#### **3. Application Setup**  
- Two Flask-based applications:  
  - **`Adiam`** (serves on port `80`)  
  - **`GetEmp`** (serves on port `8000`)  
- **Hardcoded IP addresses** in the application (temporary, should be parameterized).  
- **Database dependency**: Uses an **RDS MySQL instance** (endpoint: `port 3306`).  

---

#### **4. Docker Image Creation & Upload**  
- **Dockerfile Structure**:  
  - Uses **Ubuntu** base image.  
  - Copies `requirements.txt` → Installs dependencies (`pip install -r requirements.txt`).  
  - Copies Python files (`app.py`) and templates (`HTML` files).  
  - Exposes ports (`80` for `Adiam`, `8000` for `GetEmp`).  
  - Runs the Flask app (`CMD ["python", "app.py"]`).  

- **Steps**:  
  1. **Build images**:  
     ```sh
     docker build -t adiam:latest .
     docker build -t getemp:latest .
     ```  
  2. **Push to ECR (Elastic Container Registry)**:  
     - Created repositories (`adiam`, `getemp`).  
     - Pushed images using `docker push`.  

---

#### **5. ECS Task Definition**  
- **Task Role**: `employapp-task-role` (full DB access).  
- **Networking Mode**: Initially set to **`awsvpc`** (VPC mode).  
- **Containers Added**:  
  - `Adiam` (port `80`)  
  - `GetEmp` (port `8000`)  
- **Environment Variables**:  
  - `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` (for DB connection).  

---

#### **6. Running the Task**  
- **Issue**: Application **not accessible** despite containers running.  
  - **Reason**: **`awsvpc` networking mode** hides container ports.  
  - **Solution**: Switch to **`host` network mode** (exposes ports directly on EC2).  

- **Fix**:  
  1. Updated task definition to use **`host` networking**.  
  2. Containers now bind to host ports (`80` and `8000`).  
  3. Application becomes accessible via EC2 instance IP.  

---

#### **7. Testing the Application**  
- **`Adiam`**:  
  - Endpoint: `http://<EC2_IP>:80`  
  - **Function**: Inserts employee data into DB.  
- **`GetEmp`**:  
  - Endpoint: `http://<EC2_IP>:8000`  
  - **Function**: Retrieves employee data.  

- **Bug Fix**:  
  - **SQL Error**: Column name mismatch (`emp_id` vs `empID`).  
  - Updated Python code to match DB schema.  

---

#### **8. Key Takeaways**  
- **Networking Modes Matter**:  
  - Use **`host`** for direct port access (testing).  
  - Use **`awsvpc`** for production (isolated networking).  
- **Avoid Hardcoding**:  
  - IPs, DB credentials, and ports should be **environment variables**.  
- **Debugging Tips**:  
  - Check **container logs** (`docker logs`).  
  - Verify **port mappings** and **security groups**.  

---

### **Final Notes**  
- Successfully deployed a **multi-container Flask app** on ECS.  
- Demonstrated **networking mode impact** on accessibility.  
- Highlighted **debugging steps** for containerized apps.  

🚀 **Next Steps**:  
- Replace hardcoded values with **parameter store/Secrets Manager**.  
- Use **ALB (Application Load Balancer)** for scalable access.  
- Automate deployments with **CI/CD pipelines**.  

--- 

Let me know if you'd like any section expanded!


***################################################################################***

**31**

**################################################################################**
***################################################################################***

**31**

**################################################################################**


