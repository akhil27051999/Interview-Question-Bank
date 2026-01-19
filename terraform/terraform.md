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

#### One-Line Summary:
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

#### One-Line Summary

**Terraform workflow allows SREs to manage infrastructure changes safely by defining infrastructure as code, reviewing changes through plans, and applying them in a controlled and automated manner.**

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

**One-Line Summary**

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

### Q6 — What is the difference between `terraform plan` and `terraform apply`?
**Answer**

- `terraform plan`: Shows what changes will be made (dry-run).  
- `terraform apply`: Implements the changes.  

Best practice: Run `plan` first to review changes before applying.

---

### Q7 — How do you manage Terraform state in a team environment?
**Answer**

- Use remote state backends (S3, Azure Blob, GCS).  
- Enable state locking (DynamoDB for S3 backend, Azure Table, etc.).  
- Use workspaces or separate states per environment.  
- Apply access controls on state storage.  
- Back up state files regularly.

---

### Q8 — What are Terraform modules and why use them?
**Answer**

Modules are containers for multiple resources used together.

Benefits:
- Reusability across projects.  
- Encapsulation and abstraction of infrastructure patterns.  
- Easier organization of complex infra.  
- Versioning and sharing (private/public registries).

---

## Category 3: Advanced Features

### Q9 — What are Terraform workspaces and when to use them?
**Answer**

Workspaces allow multiple distinct states from the same configuration.

Use cases:
- Environment separation (dev/stage/prod).  
- Isolating feature-branch deployments.  
- Managing multiple regions with the same code.

---

### Q10 — Explain Terraform provisioners and their types.
**Answer**

Provisioners execute scripts or actions on resource creation/destruction.

Types:
- **local-exec**: Run a command on the machine running Terraform.  
- **remote-exec**: Run commands on the created resource (via SSH/WinRM).  
- **file**: Copy files to created resources.

Note: Use provisioners only when necessary; prefer configuration management tools for in-guest configuration.

---

### Q11 — What is `terraform import` and when is it used?
**Answer**

`terraform import` imports existing infrastructure into Terraform state.

Use cases:
- Bringing manually created resources under Terraform management.  
- Recovering resources into a state file.  
- Adopting Terraform for existing environments.

---

## Category 4: Troubleshooting & Real-time Scenarios

### Q12 — Terraform plan shows unexpected changes. How to debug?
**Answer**

- Run `terraform refresh` to sync state with real infrastructure.  
- Inspect current state with `terraform show`.  
- Review recent configuration changes.  
- Check provider/plugin version changes.  
- Use `-refresh=false` on `plan` to isolate config-only changes.

---

### Q13 — `terraform apply` fails with state locking error. How to resolve?
**Answer**

- Identify who holds the lock (team/CI).  
- Verify connectivity to remote backend.  
- Wait or coordinate with the lock holder.  
- If safe, manually remove the lock (e.g., delete lock item in DynamoDB) — only after team agreement.  
- Use `-lock-timeout` for long-running operations.

---

### Q14 — How do you handle secrets in Terraform?
**Answer**

- Avoid hardcoding secrets in `.tf` files.  
- Use environment variables for sensitive values.  
- Integrate with secret managers (HashiCorp Vault, AWS Secrets Manager).  
- Store state in encrypted remote storage.  
- Limit IAM permissions and use least privilege.

---

### Q15 — Terraform is destroying and recreating resources unnecessarily. Why?
**Answer**

Common causes:
- Changed immutable attributes (e.g., an AMI ID).  
- Computed or drifted attributes cause diffs.  
- State out of sync with real resources.  
- Provider bugs or version changes.  
- Use `terraform state` inspection and `terraform plan` to identify exact causes.

---

### Q16 — How to rollback Terraform changes?
**Answer**

- Revert code in version control and re-run `terraform apply`.  
- Use `terraform state` subcommands to manipulate state if needed.  
- Implement blue/green or canary patterns for services to avoid destructive rollbacks.  
- Keep state backups for recovery.

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
