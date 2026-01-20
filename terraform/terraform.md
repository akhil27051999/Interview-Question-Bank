# DevOps Interview Preparation Guide — Terraform

## Table of Contents
- [Category 1: Terraform Fundamentals](#category-1-terraform-fundamentals)
- [Category 2: Configuration & Modules](#category-2-configuration--modules)
- [Category 3: Advanced Features](#category-3-advanced-features)
- [Category 4: Troubleshooting & Real-time Scenarios](#category-4-troubleshooting--real-time-scenarios)
- [Category 5: Best Practices & Patterns](#category-5-best-practices--patterns)
- [Category 6: Advanced Scenarios](#category-6-advanced-scenarios)
- [Category 7: Real-world Implementation](#category-7-real-world-implementation)


# Category 1: Terraform Fundamentals

## Q1 — What is Terraform and How Does It Differ from Ansible

**Terraform**
- Terraform is an Infrastructure as Code (IaC) tool used to provision, manage, and version infrastructure declaratively.
- It ensures desired state is maintained by tracking infrastructure via a state file.
- Works with cloud providers like AWS, Azure, GCP, and on-prem.

**Ansible**
- Ansible is a configuration management and automation tool.
- Focuses on configuring software, installing packages, and managing servers.
- Uses imperative/playbook-based approach, executes tasks in order.

| Feature          | Terraform                     | Ansible                              |
| ---------------- | ----------------------------- | ------------------------------------ |
| Purpose          | Provision infrastructure      | Configure and manage software        |
| Approach         | Declarative (state-based)     | Imperative (task-based)              |
| State Management | Maintains state file          | No persistent state (optional facts) |
| Idempotency      | Built-in via state comparison | Achieved via playbook design         |
| Example Use Case | Create VPC, EC2, S3           | Install Nginx, update configs        |

### Example

**Terraform: Create an EC2 instance**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}
```

**Ansible: Install Nginx on EC2**
```yaml
- name: Install Nginx
  hosts: web
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

### One-Line Summary:
- Terraform provisions infrastructure declaratively, while Ansible configures and manages it imperatively.

---

## Q2 — Explain Terraform Workflow

- Terraform workflow is a structured process to safely provision and manage infrastructure as code using a declarative approach.
- In an SRE model, the workflow emphasizes change safety, reviewability, state management, and automation to ensure infrastructure reliability.

**The workflow consists of these main steps:**
  - `Write` – Define infrastructure in Terraform configuration files (.tf) using HCL.
  - `Init` – Initialize Terraform to download providers, modules, and configure backend state.
  - `Validate & Format` – Ensure syntax correctness and enforce coding standards.
  - `Plan` – Generate an execution plan to preview infrastructure changes before applying them.
  - `Apply` – Provision or update infrastructure based on the approved plan.
  - `State Management` – Track real infrastructure using a state file, typically stored remotely.
  - `Drift Detection` – Regularly run terraform plan to detect manual or unintended changes.

 This workflow ensures predictable, auditable, and reversible infrastructure changes, which aligns with SRE principles.

### Example

**Step 1️: Write Infrastructure Code**

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcdef12345"
  instance_type = "t3.micro"

  tags = {
    Name = "terraform-web"
  }
}
```
- Infrastructure is now defined as code.

**Step 2: Initialize Terraform**
```hcl
terraform init
```

- Downloads AWS provider
- Configures backend
- Prepares the working directory

**Step 3: Validate Configuration**
```hcl
terraform validate
terraform fmt
```
- Ensures correct syntax
- Applies standard formatting

**Step 4: Plan the Changes**
```hcl
terraform plan
```

**Output shows:**
- `+ create aws_instance.web`
- No resources destroyed
- This step is critical in production to review impact before deployment.

**Step 5: Apply the Plan**
```hcl
terraform apply
```

- Creates EC2 instance
- Updates Terraform state

**Step 6: Modify Infrastructure (Real-World Change)**

**Change instance type:**
```hcl
instance_type = "t3.small"
```

**Run:**
```hcl
terraform plan
terraform apply
```

- Terraform updates only what changed
- No unnecessary resource recreation

**Step 7: Drift Detection Example**

- If someone manually stops or changes the EC2 instance in AWS Console:
```hcl
terraform plan
```

- Terraform detects drift
- Shows differences from declared state

### One-Line Summary

Terraform workflow allows SREs to manage infrastructure changes safely by defining infrastructure as code, reviewing changes through plans, and applying them in a controlled and automated manner.

---

## Q3 — What is Terraform State and Why Is It Important?

- Terraform state is a file that stores the mapping between Terraform configuration and real-world infrastructure resources.
- It allows Terraform to understand what resources already exist, their current attributes, and what changes are required to reach the desired state.
- Terraform state is critical because it enables safe infrastructure changes, drift detection, consistency, and automation.

### Example

**You create an EC2 instance using Terraform.**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-xyz"
  instance_type = "t3.micro"
}
```
- After `terraform apply`:
  - Terraform stores the EC2 instance ID in the state file

- If you later change:

  ```hcl 
  instance_type = "t3.small"
  ```

**Terraform:**
- Checks state
- Identifies the same instance
- Updates only the instance type

**Note:** 
- Without state, Terraform wouldn’t know which EC2 instance to modify.

---

## Q4 — Explain Terraform Core Concepts: Providers, Resources, Data Sources

Providers are plugins that allow Terraform to interact with external services like AWS, Azure, or GCP.
Resources define infrastructure components that Terraform creates and manages.
Data Sources are used to fetch existing infrastructure information without creating anything.

### Example

#### Provider (Connect to AWS)

```hcl
provider "aws" {
  region = "ap-south-1"
}
```
- Tells Terraform which cloud and region to use.

#### Resource (Create Infrastructure)

```hcl
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}
```
- Terraform creates and manages this EC2 instance.

#### Data Source (Read Existing Infrastructure)
```hcl
data "aws_ami" "latest_amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
}
````
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.latest_amazon_linux.id
  instance_type = "t3.micro"
}
```

- Terraform reads existing AMI info and uses it, without creating a new AMI.

### One-Line Summary

Providers connect Terraform to cloud services, resources create and manage infrastructure, and data sources read existing infrastructure details.

---

# Category 2: Configuration & Modules

## Q5 — Terraform Variables and How They Are Used

- Terraform variables allow you to parameterize your infrastructure, making it reusable, flexible, and environment-independent.
- Variables can store values like instance types, AMIs, regions, or tags, instead of hardcoding them in .tf files.

#### Types of Variables
  1. String – Single value
  2. Number – Numeric value
  3. Boolean – True/False
  4. List / Map – Multiple values for looping or mapping

#### How They Are Used

**1. Define a variable (variables.tf):**

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t3.micro"
}

```

#### 2. Use the variable (main.tf):

```sh
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = var.instance_type
}
```
#### 3.Set variable values

 - CLI: terraform apply -var="instance_type=t3.small"
 - TFVARS file: terraform.tfvars

```hcl
instance_type = "t3.small"
```

### Example
- `Scenario`: Deploy multiple environments (dev, prod) with different instance types.
```hcl
# variables.tf
variable "env" {
  description = "Deployment environment"
}

variable "instance_type" {
  description = "EC2 instance type"
  default     = "t3.micro"
}

# main.tf
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = var.instance_type
  tags = {
    Environment = var.env
  }
}
```

- For dev: `terraform apply -var="env=dev" -var="instance_type=t3.micro"`
- For prod: `terraform apply -var="env=prod" -var="instance_type=t3.small"`

### One-Line Summary

Terraform variables make your code dynamic and reusable, allowing SREs to manage multiple environments safely without changing the core configuration.

---

## Q6 — Difference Between terraform plan and terraform apply

- terraform plan shows a preview of changes Terraform will make without actually modifying infrastructure.

- terraform apply executes those changes and updates real infrastructure and the state file.

| Aspect       | terraform plan    | terraform apply    |
| ------------ | ----------------- | ------------------ |
| Action       | Preview only      | Makes real changes |
| Infra Change |  No               |  Yes               |
| State Update |  No               |  Yes               |
| Use Case     | Review & approval | Deploy changes     |

### Example

**Change EC2 instance type in code:**
```hcl
instance_type = "t3.small"
```
```hcl
terraform plan
```

- Shows: `~ update aws_instance.web`
```hcl
terraform apply
```
- Updates EC2 instance
- Updates Terraform state

### One-Line Summary

terraform plan helps review and prevent risky changes, while terraform apply safely executes approved infrastructure changes.

---

## Q7 — How Do You Manage Terraform State in a Team Environment?

In a team environment, Terraform state is managed using a remote backend with state locking and access control to avoid conflicts and ensure consistency.

**Key Practices**

- Remote State – Store state in S3 / GCS / Terraform Cloud
- State Locking – Prevents simultaneous updates (e.g., DynamoDB)
- Access Control – Restrict state access using IAM roles
- Versioning & Backups – Enables recovery from state corruption
- CI/CD Pipelines – Run Terraform via pipelines, not local machines

### Example (AWS)

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "app/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

#### This setup:

- Stores state centrally
- Locks state during apply
- Allows safe collaboration

### One-Line Summary

Terraform state in teams is managed using remote backends with locking, access control, and CI/CD to ensure safe and consistent infrastructure changes.

---

## Q8 — What Are Terraform Modules and Why Use Them?

- Terraform modules are reusable, self-contained collections of Terraform files that define a set of related resources.
- They help teams avoid duplication, standardize infrastructure, and manage complex setups easily.

**Why Use Modules ?**

- Reusability across environments (dev, prod)
- Cleaner and more maintainable code
- Standardized best practices
- Easier scaling and updates

### Example

**Create a module (modules/ec2/main.tf):**

```hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

**Use the module (main.tf):**
```hcl
module "web_ec2" {
  source        = "./modules/ec2"
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}
```

- Same module can be reused for dev, staging, prod with different values.

### One-Line Summary

Terraform modules allow teams to build reusable, standardized infrastructure components, improving maintainability and consistency.

---

# Category 3: Advanced Features

## Q9 — What Are Terraform Workspaces and When to Use Them?

Terraform workspaces allow you to manage multiple state files using the same Terraform configuration.
Each workspace represents a separate environment with its own state.

**When to Use Workspaces**
- Managing multiple environments (dev, staging, prod) with the same code
- Isolating state without duplicating Terraform configs
- Simple environment separation

Not recommended for large or highly critical prod environments (better use separate backends).

### Example

**Create and switch workspaces:**
```hcl
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
```

**Use workspace in code:**
```hcl
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.small" : "t3.micro"
}
```
- Each workspace has its own state file, but uses the same configuration.

### One-Line Summary

Terraform workspaces let you reuse the same code for multiple environments by keeping separate state files.

---
## Q10 — Explain Terraform Provisioners and Their Types

- Terraform provisioners are used to execute scripts or commands on a resource after it is created or before it is destroyed.
- They are mainly used for bootstrapping and last-mile configuration.
- Provisioners should be used sparingly; configuration tools like Ansible, cloud-init, or user data are preferred.

**Types of Provisioners**

1. file – Copies files to a resource
2. local-exec – Runs commands on the machine running Terraform
3. remote-exec – Runs commands on the created resource

### Example

**remote-exec (installing Nginx on EC2):**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
  }
}
```
- After EC2 is created, Terraform connects via SSH and installs Nginx.

### One-Line Summary

Terraform provisioners run scripts during resource lifecycle but should be avoided in favor of configuration management tools for production reliability.

---

## Q11 — What Is terraform import and When Is It Used?

- terraform import is used to bring existing infrastructure resources under Terraform management without recreating them.
- It maps an already-created resource to Terraform state.

**When to Use `terraform import`**
- Infrastructure was created manually (console / CLI)
- Migrating legacy infrastructure to Terraform
- Adopting Terraform in an existing environment
- Import updates only the state, not the .tf code.

### Example

An EC2 instance already exists in AWS with ID `i-0abc123`.

**Define the resource:**
```hcl
resource "aws_instance" "web" {}
```

**Import it:**
```bash
terraform import aws_instance.web i-0abc123
```
- Terraform now tracks the EC2 instance in state.

**Next step:**
```bash
terraform plan
```

- Shows differences
- Helps align code with real infrastructure

### One-Line Summary

terraform import allows existing resources to be managed by Terraform by adding them to the state without recreating them.

---

## Category 4: Troubleshooting & Real-time Scenarios

## Q12 — Terraform Plan Shows Unexpected Changes. How Do You Debug?

When Terraform plan shows unexpected changes, I debug by checking state, drift, configuration differences, and provider behavior to identify why Terraform thinks a change is needed.

### Debugging Steps (Practical)

#### 1. Check for Drift

- Verify if someone changed resources manually in the console

```hcl
terraform plan
```

#### 2. Review Recent Code Changes

- Compare .tf files with the last known good version (Git diff)

#### 3. Inspect State

- Check what Terraform believes exists:

```hcl
terraform state show <resource>
```

#### 4. Check Provider or Version Changes

- Provider upgrades can change defaults or behavior
- Lock provider versions

#### 5. Look for Dynamic Values

- Attributes like timestamps, AMIs, or auto-generated values can cause diffs
- Use lifecycle `{ ignore_changes = [...] }` if needed

#### 6. Run Refresh (if needed)
```hcl
terraform refresh
```
### Example

- Terraform plan shows EC2 instance replacement.
  - Check if AMI changed
  - Check if user data was modified
  - Verify if change requires force recreation

### One-Line Summary

Unexpected Terraform plan changes are usually caused by drift, state mismatch, or provider changes, and I debug them by inspecting state, code, and provider behavior.

---

## Q13 — terraform apply Fails with State Locking Error. How Do You Resolve It?

A state locking error happens when Terraform detects that the state file is already locked, usually because another apply is running or a previous run didn’t release the lock.
To resolve it, I identify the lock owner and safely unlock the state.

### Resolution Steps (Practical)

#### 1. Check Who Holds the Lock
  - Terraform output shows lock ID, user, and time
  - Confirm no active terraform apply is running

#### 2. Wait or Coordinate
  - If another teammate or CI job is running, wait for it to finish

#### 3. Force Unlock (Only If Safe)
```hcl
terraform force-unlock <LOCK_ID>
```
- Use only when you’re sure no apply is in progress

#### 4. Verify Backend Locking
- Check DynamoDB table (AWS) or backend health
- Ensure permissions are correct

#### 5. Retry Apply
```hcl
terraform apply
```

### Example (AWS S3 + DynamoDB)

- A CI pipeline crashed during apply → lock remained in DynamoDB.
  - Confirm pipeline stopped
    ```hcl
    terraform force-unlock d4f3c9b2
    ```
  - Re-run pipeline safely

### One-Line Summary

State locking errors are resolved by identifying the lock owner and safely releasing the lock, usually with terraform force-unlock, after confirming no active apply is running.

---

## Q14 — How Do You Handle Secrets in Terraform?

Secrets in Terraform should never be hardcoded. They are handled using secure secret managers, environment variables, and sensitive variables, while keeping them out of state files and version control as much as possible.

### Best Practices (Practical)

#### 1. Use Secret Managers
  - AWS Secrets Manager / SSM Parameter Store / Vault
  - Fetch secrets using data sources

#### 2. Mark Variables as Sensitive
```hcl
variable "db_password" {
  sensitive = true
}
```
#### 3. Use Environment Variables
```hcl
export TF_VAR_db_password="mypassword"
```

#### 4. Secure the State File
- Remote backend (S3)
- Encryption at rest
- Restricted IAM access

#### 5. Avoid Outputting Secrets
```hcl
output "db_password" {
  value     = var.db_password
  sensitive = true
}
```

### Example (AWS SSM)
```hcl
data "aws_ssm_parameter" "db_password" {
  name            = "/prod/db/password"
  with_decryption = true
}
```
```hcl
resource "aws_db_instance" "db" {
  password = data.aws_ssm_parameter.db_password.value
}
```

### One-Line Summary

Secrets in Terraform are handled using external secret managers, sensitive variables, and secure state backends, never by hardcoding values.

---

## Q15 — Terraform Is Destroying and Recreating Resources Unnecessarily. Why?

Terraform destroys and recreates resources when it detects a change in an attribute that is marked as ForceNew, meaning the provider requires resource replacement instead of an in-place update.

### Common Real-World Reasons

**1. ForceNew Attributes Changed**
  - Example: AMI ID, subnet, availability zone
  - These changes require recreation by the provider

**2. Immutable Infrastructure Design**
  - Some resources are intentionally replaced (e.g., Launch Templates)

**3. State Drift**
  - Manual changes outside Terraform cause mismatch

**4. User Data Changes (EC2)**
  - Modifying user_data often forces replacement

**5. Resource Name / ID Changes**
  - Changing unique identifiers triggers recreation

### Real Practice Example

```hcl
ami = "ami-new123"
```
```sh
terraform plan
```

**Output:**
```hcl
-/+ aws_instance.web (forces new resource)
```
- Terraform must destroy and recreate the instance.

### How to Reduce Unnecessary Recreation
- Review terraform plan carefully
- Use:
  ```hcl
  lifecycle {
    ignore_changes = [user_data]
  }
  ```
  - Avoid manual changes
  - Use modules with stable inputs

### One-Line Summary

Terraform recreates resources when immutable or ForceNew attributes change, or when drift causes state mismatch.

---

## Q16 — How to Rollback Terraform Changes

Terraform doesn’t have a built-in rollback command. Rollback is achieved by reverting the configuration to the previous state and re-applying it, using version control and state backups.

### Practical Steps

#### 1. Revert Code in Git
```sh
git checkout <previous-commit>
```

#### 2. Use Previous State (Optional)
  - Remote backend keeps state versions
  - Restore previous state if needed

#### 3. Apply Old Configuration
```sh
terraform apply
```
  - Terraform aligns real infra with the reverted configuration

#### 4. Handle Destructive Changes Carefully
  - Use terraform plan to confirm changes
  - Optionally use lifecycle { prevent_destroy = true } on critical resources

### Example
- Accidentally increased EC2 instance type in prod (t3.micro → t3.large)
- Revert code to original t3.micro
- Run:
  ```sh
  terraform plan
  terraform apply
  ```
- EC2 instance is resized back to safe configuration

### One-Line Summary

Rollback in Terraform is done by reverting code/state to a previous version and re-applying it, ensuring infrastructure matches the known safe state.

---

# Category 5: Best Practices & Patterns

## Q17 — What Is Terraform Backend Configuration and Why Is It Important?

A Terraform backend defines where Terraform stores its state file and how it performs operations.
It’s important because it ensures state consistency, team collaboration, and secure storage of the source-of-truth for infrastructure.

### Key Points
- Local backend – Default, stores state locally (not ideal for teams)
- Remote backend – Stores state in S3, GCS, Terraform Cloud, etc.
- Supports:
  - State locking (prevents concurrent updates)
  - Versioning & backup
  - Encryption at rest
  - CI/CD integration

### Example (AWS S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "app/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

- Ensures remote storage, locking, and encrypted state for safe team collaboration.

### One-Line Summary

Terraform backend stores state remotely to enable safe collaboration, locking, versioning, and secure infrastructure management.

---

## Q18 — How Do You Structure Large Terraform Projects?

Large Terraform projects are structured using modules, workspaces, and environment separation to improve reusability, maintainability, and scalability.

### Best Practices
#### 1. Use Modules for Reusable Components
  - Example: VPC, EC2, RDS, Load Balancer
  - Keeps code DRY and consistent

#### 2. Separate Environments
- Use folders or workspaces for dev, staging, prod
- Example:
```txt
├── environments
│   ├── dev
│   ├── staging
│   └── prod
└── modules
    ├── vpc
    ├── ec2
    └── rds
```

#### 3. Use Remote Backends
  - Store state centrally (S3/GCS/Terraform Cloud)
  - Enable locking and versioning

#### 4. Use Variable and TFVARS Files
  - Parameterize modules and environment differences

#### 5. CI/CD Integration
  - Automate terraform plan and terraform apply in pipelines
  - Ensure peer review and approval

### Example

- Module: `modules/ec2/main.tf`
- Environment: `environments/prod/main.tf`
```hcl
module "web_server" {
  source        = "../../modules/ec2"
  instance_type = var.instance_type
  ami           = var.ami
}
```
- CI/CD runs terraform plan and apply for prod workspace, dev workspace, etc.

### One-Line Summary

Large Terraform projects are structured with modules, environment separation, variables, and remote backends to ensure maintainable, reusable, and team-friendly infrastructure.

---

## Q19 — What Is Terraform Cloud/Enterprise and Its Benefits

Terraform Cloud (or Enterprise) is a managed service for Terraform that provides remote state management, collaboration, policy enforcement, and CI/CD automation.
It helps teams scale infrastructure safely and reliably.

### Key Benefits
- **Remote State Management** – Centralized state storage with versioning and locking
- **Collaboration** – Team-based access controls and shared workspaces for dev/staging/prod
- **Policy as Code** – Use Sentinel policies to enforce compliance before `apply`
- **Automated Workflows** – Plan and apply via VCS triggers (GitHub, GitLab, etc.)
- **Audit & Governance** – Track who applied changes and when

### Real Practice Example
1. Team stores state in Terraform Cloud
2. Developers create a PR → `terraform plan` runs automatically
3. After approval, `terraform apply` executes in remote workspace
4. No manual state handling; all changes are audited

### One-Line Summary
Terraform Cloud/Enterprise enables secure, collaborative, and automated infrastructure management with remote state, policy enforcement, and CI/CD integration.

---

## Q20 — How Do You Implement Dependency Management in Terraform?

Terraform automatically manages dependencies using a resource dependency graph, but explicit dependencies can be defined with depends_on when necessary.
This ensures resources are created, updated, or destroyed in the correct order.

### How Terraform Handles Dependencies

#### 1. Implicit Dependencies
- Terraform detects dependencies from references between resources.
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.latest.id
  instance_type = "t3.micro"
}
```
- Here, `aws_instance.web` depends on `data.aws_ami.latest` automatically.

#### 2. Explicit Dependencies
- Use depends_on when Terraform can’t infer order.
```hcl
resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.main.id
  name    = "www.example.com"
  type    = "A"
  ttl     = 300
  records = [aws_instance.web.public_ip]

  depends_on = [aws_instance.web]
}
```
### Example

- Create VPC → Subnet → EC2
- Subnet must exist before EC2
- Terraform handles implicitly via `vpc_id` reference:
```hcl
resource "aws_subnet" "app" {
  vpc_id = aws_vpc.main.id
}
resource "aws_instance" "web" {
  subnet_id = aws_subnet.app.id
}
```
- EC2 will automatically wait until Subnet is created.

### One-Line Summary

Terraform manages dependencies automatically via references and the dependency graph, and depends_on can be used for explicit ordering when needed.

---

# Category 6: Advanced Scenarios


## Q21 — What Are Terraform Data Sources and When to Use Them?

Terraform data sources allow you to fetch or read existing infrastructure information without creating new resources.
They are used when you need to reference existing resources in your configuration safely.

**When to Use Data Sources**
- To get details of existing resources (e.g., AMIs, subnets, VPCs)
- To avoid hardcoding values
- To ensure dynamic and environment-agnostic configurations

#### Example

**Fetch the latest Amazon Linux AMI for an EC2 instance:**
```hcl
data "aws_ami" "latest_amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.latest_amazon_linux.id
  instance_type = "t3.micro"
}

```

- Terraform reads the existing AMI ID and provisions EC2 using it, without creating a new AMI.

**One-Line Summary**

Data sources let Terraform read existing infrastructure attributes to make configurations dynamic and avoid hardcoding.

---

## Q22 — How Do You Handle Multiple Environments in Terraform?

Multiple environments (dev, staging, prod) in Terraform are handled using workspaces, folder-based separation, and parameterized variables. This ensures isolation, reusability, and safe deployments.

### Common Approaches

#### 1. Workspaces

Each workspace has a separate state file using the same code.
```hcl
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
```

#### 2. Folder-Based Environment Structure
```txt
environments/
  dev/
    main.tf
    terraform.tfvars
  prod/
    main.tf
    terraform.tfvars
modules/
  vpc/
  ec2/
```
#### 3. Variable Files (TFVARS)
- Environment-specific variables passed at apply:
```hcl
terraform apply -var-file="environments/dev/terraform.tfvars"
```

#### 4.Parameterized Modules
- Reuse same modules with different inputs per environment.

#### Example
```hcl
# environments/prod/terraform.tfvars
instance_type = "t3.large"
vpc_id        = "vpc-abc123"

# environments/dev/terraform.tfvars
instance_type = "t3.micro"
vpc_id        = "vpc-def456"
```
```sh
terraform apply -var-file="environments/dev/terraform.tfvars"
```
- Dev and prod use the same code but isolated infrastructure and state.

### One-Line Summary

Multiple environments in Terraform are handled via workspaces, folder separation, and parameterized variables to isolate and safely manage infrastructure per environment.

---

## Q23 — What Are terraform taint and terraform untaint?
- `terraform taint` marks a resource as damaged or needing replacement. Terraform will destroy and recreate it on the next apply.
- `terraform untaint` removes the tainted status so Terraform will not recreate the resource.

### Example

#### 1. Taint a Resource
- Suppose an EC2 instance is misconfigured:
```hcl
terraform taint aws_instance.web
```
- Next `terraform apply` will recreate this EC2 instance.

#### 2. Undo Taint
- If the resource is fine:
```hcl
terraform untaint aws_instance.web
```

- Terraform will keep the existing resource unchanged on the next apply.

### One-Line Summary

terraform taint forces a resource to be replaced, and terraform untaint reverses that action to preserve the resource.

---

## Q24 — How Do You Manage Terraform Provider Versions?

Terraform provider versions are managed using the required_providers block and version constraints to ensure reproducible and stable infrastructure across environments.
Locking provider versions prevents breaking changes when the provider updates.

### Best Practices

#### 1. Specify Provider Version
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.5.0"
}
```

#### 2. Use Version Constraints

- `=` exact version
- `~>` compatible version (patch updates allowed)
- `>=` minimum version

#### 3. Check Provider Lock File
- Terraform generates .terraform.lock.hcl to pin versions used by your project
- Commit the lock file for team consistency

#### 4. Update Carefully
```hcl
terraform init -upgrade
```
- Only update when tested to avoid breaking infrastructure

### Example

**Team project uses AWS provider:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```
- .terraform.lock.hcl ensures all team members use AWS provider v5.x, avoiding drift or incompatibility.

### One-Line Summary

Terraform provider versions are managed with required_providers and lock files to ensure stable, reproducible, and team-safe infrastructure deployments.

---

## Q25 — What Is Terraform State Locking and Why Is It Crucial?

- Terraform state locking prevents simultaneous modifications to the state file by multiple users or CI/CD pipelines.
- It is crucial to avoid race conditions, corrupted state, and accidental resource destruction in team environments.

### How It Works
- Remote backends like S3 + DynamoDB, Terraform Cloud, or GCS support locking.
- When a user runs terraform apply, Terraform acquires a lock on the state.
- Other users or processes cannot apply changes until the lock is released.

### Example (AWS S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "app/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
- DynamoDB table manages state locks
- Prevents multiple pipelines from applying changes at the same time

### One-Line Summary

Terraform state locking ensures safe, concurrent team operations by preventing simultaneous modifications to the infrastructure state.

---

# Category 7: Real-world Implementation

## Q26 — How Do You Implement CI/CD with Terraform?

CI/CD with Terraform automates infrastructure provisioning, review, and deployment, ensuring repeatable, safe, and auditable changes.

### Key Steps
#### 1. Store Terraform Code in VCS
  - GitHub, GitLab, Bitbucket, etc.
  - Enables version control and peer review
#### 2. Plan Stage (CI)
  - Triggered on pull requests or branch changes
  - Runs:
  ```hcl
  terraform init
  terraform validate
  terraform plan -out=tfplan
  ```
  - Plan is reviewed before applying

#### 3. Apply Stage (CD)
  - Triggered after approval or merge to main/prod
  - Runs:
  ```hcl
  terraform apply tfplan
  ```
  - Updates infrastructure safely and updates remote state

#### 4. Use Remote Backends and State Locking
  - Ensure safe concurrent operations

#### 5. Environment Isolation
  - Use workspaces or folder separation for dev/staging/prod

### Example

**GitHub Actions Pipeline:**
```yaml
name: Terraform CI/CD
on:
  pull_request:
    branches: [main]

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      - run: terraform init
      - run: terraform validate
      - run: terraform plan -out=tfplan

  apply:
    needs: plan
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - run: terraform apply -auto-approve tfplan
```
- Automates plan and apply, keeps state consistent, and enforces approvals.

### One-Line Summary

CI/CD with Terraform automates plan, review, and apply using version control, pipelines, and remote state to ensure safe, repeatable infrastructure changes.

---

## Q27 — What Are Terraform Dynamic Blocks?

- Terraform dynamic blocks allow you to generate multiple nested blocks programmatically based on variables, lists, or maps.
- They are useful when the number of sub-blocks or their values are not known in advance.

### Key Points
- Helps avoid repetitive code
- Works with resource arguments that accept multiple nested blocks
- Often used with for_each or count inside the dynamic block

### Example
- Suppose you want to attach multiple security group rules dynamically:
```hcl
variable "sg_rules" {
  type = list(object({
    type        = string
    from_port   = number
    to_port     = number
    cidr_blocks = list(string)
  }))
}

resource "aws_security_group" "web_sg" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.sg_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.type
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

- `var.sg_rules` can have any number of rules
- Terraform generates a corresponding ingress block for each rule

### One-Line Summary

Terraform dynamic blocks let you programmatically create nested blocks based on variables, reducing repetitive code and improving flexibility.

---

## Q28 — How Do You Handle Conditional Resource Creation?

- In Terraform, conditional resource creation is handled using the count or for_each argument with a conditional expression.
- This allows resources to be created only when certain conditions are met, making infrastructure flexible and environment-specific.

### Approaches

#### 1. Using `count`
```hcl
resource "aws_instance" "web" {
  count         = var.create_instance ? 1 : 0
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}
```
- If var.create_instance is false, no EC2 instance is created.

#### 2. Using `for_each`
```hcl
resource "aws_security_group_rule" "allow_ports" {
  for_each = var.open_ports_enabled ? toset(var.ports) : {}
  type        = "ingress"
  from_port   = each.value
  to_port     = each.value
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```
- Creates rules only if var.open_ports_enabled is true.

### Example
- Dev environment: optional EC2 instance
```hcl
variable "create_dev_ec2" {
  default = true
}

resource "aws_instance" "dev_web" {
  count         = var.create_dev_ec2 ? 1 : 0
  ami           = "ami-dev123"
  instance_type = "t3.micro"
}
```
- Prod environment: `set create_dev_ec2 = false` → no dev instance created

### One-Line Summary

Conditional resource creation in Terraform is implemented using count or for_each with conditions, allowing resources to be created only when needed.

---

## Q29 — What Is terraform graph and How Is It Useful?

- terraform graph generates a visual representation of the dependency graph of your Terraform resources.
- It is useful for understanding resource dependencies, debugging complex infrastructure, and planning changes safely.

### Key Points
- Shows how resources depend on each other
- Helps identify potential issues or order of creation
- Outputs in DOT format, which can be visualized using tools like Graphviz

### Example
- Generate the graph:
```hcl
terraform graph > graph.dot
dot -Tpng graph.dot -o graph.png
```
- This produces a PNG image showing resource relationships
- Example: VPC → Subnet → EC2 → Security Group
- Helps SREs verify dependencies and avoid accidental deletion of dependent resources

### One-Line Summary

terraform graph visualizes resource dependencies, helping teams understand complex infrastructure and plan safe changes.

---

## Q30 — How Do You Implement Disaster Recovery (DR) with Terraform?

Disaster recovery with Terraform is implemented by version-controlled infrastructure code, automated provisioning, remote state, and multi-region/environment strategies. This ensures infrastructure can be recreated quickly in case of failures.

### Key Practices

#### 1. Version Control Everything
- Store Terraform code in Git or similar VCS
- Roll back to known good state easily

#### 2. Remote State Management
- Store state in S3, GCS, or Terraform Cloud with versioning
- Enables recovery of last known infrastructure state

#### 3. Automated Provisioning
- Use Terraform scripts to recreate resources in another region or account

#### 4. Environment & Region Separation
- Maintain prod and DR environments in separate regions
- Use parameterized variables or workspaces

#### 5. CI/CD Pipelines
- Automate terraform plan and apply for DR deployment

### Example
- Scenario: Primary AWS region goes down
- DR Strategy:
  - Terraform code + modules stored in Git
  - State stored in S3 with versioning
  - Apply same code in DR region:
  ```sh
  terraform workspace select dr
  terraform apply -var="region=ap-south-1"
  ```
  - Infrastructure is recreated automatically in DR region

### One-Line Summary

Disaster recovery with Terraform is achieved by storing code and state securely, version-controlling everything, and automating infrastructure provisioning in alternate regions or environments.
