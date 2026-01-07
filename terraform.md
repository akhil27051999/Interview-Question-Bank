# DevOps Interview Preparation Guide — Terraform

## Table of Contents
- [Category 1: Terraform Fundamentals](#category-1-terraform-fundamentals)
- [Category 2: Configuration & Modules](#category-2-configuration--modules)
- [Category 3: Advanced Features](#category-3-advanced-features)
- [Category 4: Troubleshooting & Real-time Scenarios](#category-4-troubleshooting--real-time-scenarios)
- [Category 5: Best Practices & Patterns](#category-5-best-practices--patterns)
- [Category 6: Advanced Scenarios](#category-6-advanced-scenarios)
- [Category 7: Real-world Implementation](#category-7-real-world-implementation)

---

## Category 1: Terraform Fundamentals

### Q1 — What is Terraform and how does it differ from Ansible?
**Answer**

Terraform is an Infrastructure as Code (IaC) tool for building, changing, and versioning infrastructure safely and efficiently.

Differences:
- **Terraform**: Declarative, focuses on infrastructure provisioning, state management.  
- **Ansible**: Procedural, focuses on configuration management, agentless.

---

### Q2 — Explain Terraform workflow.
**Answer**

Typical workflow:
1. **Write**: Define resources in `.tf` files.  
2. **Plan**: `terraform plan` to preview changes.  
3. **Apply**: `terraform apply` to create/update resources.  
4. **Destroy**: `terraform destroy` to remove resources.

---

### Q3 — What is Terraform state and why is it important?
**Answer**

The state file (`terraform.tfstate`) tracks resource metadata and relationships.

Importance:
- Maps real-world resources to configuration.  
- Tracks resource dependencies.  
- Stores resource attributes for performance.  
- Enables collaboration when stored remotely.

---

### Q4 — Explain Terraform core concepts: Providers, Resources, Data Sources.
**Answer**

- **Providers**: Plugins that interact with APIs (AWS, Azure, GCP).  
- **Resources**: Infrastructure components to manage (e.g., `aws_instance`, `aws_s3_bucket`).  
- **Data Sources**: Read-only lookups for existing infrastructure data.

---

## Category 2: Configuration & Modules

### Q5 — What are Terraform variables and how are they used?
**Answer**

Variables parameterize Terraform configurations.

Types:
- **Input variables**: Accept values from users/environment.  
- **Output values**: Return information about created infra.  
- **Local values**: Assign names to expressions for reuse.

Example:
```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

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
