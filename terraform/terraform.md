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

## Category 5: Best Practices & Patterns

### Q17 — What is Terraform backend configuration and why is it important?
**Answer**

The backend determines where state is stored and how operations are executed.

Importance:
- Remote, shared state for team collaboration.  
- State locking to prevent concurrent modifications.  
- Centralized access control and auditing.  
- Support for remote operations (Terraform Cloud/Enterprise).

Example backend snippet:
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "project/terraform.tfstate"
    region = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}
```

---

### Q18 — How do you structure large Terraform projects?
**Answer**

Common patterns:
- **Module-based**: Create reusable modules for repeated patterns.  
- **Environment separation**: Separate directories or workspaces for dev/stage/prod.  
- **Monorepo vs multi-repo**: Choose based on team and release needs.  
- **Hybrid**: Shared modules + per-environment overlays.

Keep modules focused and parameterized; use a root orchestration layer to compose modules.

---

### Q19 — What is Terraform Cloud/Enterprise and its benefits?
**Answer**

Terraform Cloud/Enterprise is a managed platform for Terraform.

Benefits:
- Remote state and runs.  
- Collaborative workspaces.  
- Policy as Code (Sentinel).  
- Private module registry.  
- Role-based access control and audit logs.  
- Integrated VCS workflows.

---

### Q20 — How do you implement dependency management in Terraform?
**Answer**

- Implicit dependencies through resource references (recommended).  
- Explicit dependencies with `depends_on` meta-argument when necessary.  
- Modules depend via input/output variables.  
- Use data sources to fetch existing resource attributes where needed.

---

## Category 6: Advanced Scenarios

### Q21 — What are Terraform data sources and when to use them?
**Answer**

Data sources read information from providers without managing creation.

Use cases:
- Lookup latest AMI IDs.  
- Get existing VPC/subnet IDs.  
- Fetch list of existing resources for referencing in new infra.

Example:
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  filter { name = "name"  value = ["ubuntu/images/*"] }
  owners = ["099720109477"]
}
```

---

### Q22 — How do you handle multiple environments in Terraform?
**Answer**

Approaches:
- **Workspaces**: Same code, separate states.  
- **Directory per environment**: Separate configs per env.  
- **Terragrunt**: Wrapper tool to DRY common config and manage env overlays.  
- **Git branches**: Branch-per-environment pattern (less recommended for state management).

Choose based on complexity, team size, and lifecycle requirements.

---

### Q23 — What is `terraform taint` and `terraform untaint`?
**Answer**

- `terraform taint <resource>`: Marks a resource for recreation on next `apply`.  
- `terraform untaint <resource>`: Removes the taint mark.  

Use case: Force recreation of a problematic resource.

---

### Q24 — How do you manage Terraform provider versions?
**Answer**

Specify provider requirements in configuration:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
```
Pin versions to avoid unexpected provider upgrades.

---

### Q25 — What is Terraform state locking and why is it crucial?
**Answer**

State locking prevents concurrent `apply` operations that would corrupt or conflict state.

Why it matters:
- Prevents state corruption and resource conflicts.  
- Ensures safe team collaboration and CI runs.  
- Backends like S3 support locking via DynamoDB; other backends have native locking.

---

## Category 7: Real-world Implementation

### Q26 — How do you implement CI/CD with Terraform?
**Answer**

Typical flow:
1. Store Terraform code in VCS (Git).  
2. Use CI to run `terraform fmt`, `terraform validate`, and `terraform plan`.  
3. Save plan artifacts and require approvals for production.  
4. Use remote state with locking.  
5. Apply in CI/CD using service principals/managed identities with least privilege.  
6. Integrate policy checks (Sentinel/OPA) and automated testing for infra.

---

### Q27 — What are Terraform dynamic blocks?
**Answer**

Dynamic blocks generate nested multiple blocks programmatically.

Example:
```hcl
dynamic "security_group_rule" {
  for_each = var.rules
  content {
    type        = security_group_rule.value.type
    from_port   = security_group_rule.value.from_port
    to_port     = security_group_rule.value.to_port
    protocol    = security_group_rule.value.protocol
    cidr_blocks = security_group_rule.value.cidr_blocks
  }
}
```

Use dynamic blocks for variable-length nested blocks (security rules, tags, etc.).

---

### Q28 — How do you handle conditional resource creation?
**Answer**

Use `count` or `for_each` with conditional expressions.

Example:
```hcl
resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0
  ami           = var.ami
  instance_type = var.instance_type
}
```

This avoids creating resources when not required.

---

### Q29 — What is `terraform graph` and how is it useful?
**Answer**

`terraform graph` outputs the dependency graph in DOT format.

Use cases:
- Visualize resource dependencies.  
- Troubleshoot complex creation/destruction order.  
- Document infra relationships.

Example:
```bash
terraform graph | dot -Tpng > graph.png
```

---

### Q30 — How do you implement disaster recovery with Terraform?
**Answer**

Practices:
- Maintain regular state backups and versioned remote state.  
- Keep all configs in version control.  
- Build modular, repeatable configs for easy recreation.  
- Document any manual/preconditions.  
- Use automated `plan`/`apply` in CI and periodically test recoveries.  
- Use infrastructure testing and drift detection.
