# DevOps Interview Preparation Guide

## Table of Contents
- [Terraform](#terraform)
  - [Category 1: Terraform Fundamentals](#category-1-terraform-fundamentals)
  - [Category 2: Configuration & Modules](#category-2-configuration--modules)
  - [Category 3: Advanced Features](#category-3-advanced-features)
  - [Category 4: Troubleshooting & Real-time Scenarios](#category-4-troubleshooting--real-time-scenarios)
  - [Category 5: Best Practices & Patterns](#category-5-best-practices--patterns)
  - [Category 6: Advanced Scenarios](#category-6-advanced-scenarios)
  - [Category 7: Real-world Implementation](#category-7-real-world-implementation)

---

## Terraform

### Category 1: Terraform Fundamentals

1. **What is Terraform and how does it differ from Ansible?**  
Answer:  
Terraform is an Infrastructure as Code (IaC) tool for building, changing, and versioning infrastructure safely and efficiently.

Differences:
- **Terraform**: Declarative, focuses on infrastructure provisioning, state management.  
- **Ansible**: Procedural, focuses on configuration management, agentless.

2. **Explain Terraform workflow.**  
Answer:
- **Write**: Define resources in .tf files.  
- **Plan**: `terraform plan` to see execution plan.  
- **Apply**: `terraform apply` to create resources.  
- **Destroy**: `terraform destroy` to remove resources.

3. **What is Terraform state and why is it important?**  
Answer:  
State file (`terraform.tfstate`) tracks resource metadata and relationships.

Importance:
- Maps real-world resources to configuration.  
- Tracks resource dependencies.  
- Enables performance (stores resource attributes).  
- Allows collaboration when stored remotely.

4. **Explain Terraform core concepts: Providers, Resources, Data Sources.**  
Answer:
- **Providers**: Plugins that interact with APIs (AWS, Azure, GCP).  
- **Resources**: Infrastructure components to manage (EC2 instance, S3 bucket).  
- **Data Sources**: Read-only information about existing infrastructure.

---

### Category 2: Configuration & Modules

5. **What are Terraform variables and how are they used?**  
Answer:  
Variables parameterize Terraform configurations.

Types:
- **Input variables**: Parameterize configurations.  
- **Output values**: Return information about infrastructure.  
- **Local values**: Assign names to expressions.

6. **What is the difference between terraform plan and terraform apply?**  
Answer:
- `terraform plan`: Shows what changes will be made (dry-run).  
- `terraform apply`: Actually implements the changes.  
Best Practice: Always run `plan` before `apply`.

7. **How do you manage Terraform state in a team environment?**  
Answer:
- Use remote state storage (S3, Azure Storage, GCS).  
- Implement state locking (DynamoDB, Azure Table Storage).  
- Use workspaces for environment separation.  
- Implement access controls for state files.

8. **What are Terraform modules and why use them?**  
Answer:  
Modules are containers for multiple resources that are used together.

Benefits:
- Reusability across projects.  
- Organization of complex infrastructure.  
- Abstraction of implementation details.  
- Versioning and sharing.

---

### Category 3: Advanced Features

9. **What are Terraform workspaces and when to use them?**  
Answer:  
Workspaces allow managing multiple distinct states with the same configuration.

Use Cases:
- Environment separation (dev, staging, prod).  
- Feature branch deployments.  
- Regional deployments with same configuration.

10. **Explain Terraform provisioners and their types.**  
Answer:  
Provisioners execute scripts on resource creation/destruction.

Types:
- **local-exec**: Run script on machine running Terraform.  
- **remote-exec**: Run script on created resource.  
- **file**: Copy files to created resource.

11. **What is terraform import and when is it used?**  
Answer:  
Command to import existing infrastructure into Terraform state.

Use Cases:
- Migrating manually created infrastructure to Terraform management.  
- Disaster recovery of state file.  
- Adopting Terraform in existing environments.

---

### Category 4: Troubleshooting & Real-time Scenarios

12. **Terraform plan shows unexpected changes. How to debug?**  
Answer:
- Run `terraform refresh` to sync state.  
- Use `terraform show` to inspect current state.  
- Review recent configuration changes.  
- Check provider version updates.  
- Use `-refresh=false` to isolate configuration vs. state changes.

13. **Terraform apply fails with state locking error. How to resolve?**  
Answer:
- Check who holds the lock.  
- Verify network connectivity to state backend.  
- Manually remove lock if safe (consult team).  
- Implement proper state locking configuration.  
- Use `-lock-timeout` for long-running operations.

14. **How do you handle secrets in Terraform?**  
Answer:
- Use environment variables for sensitive values.  
- Integrate with secret managers (HashiCorp Vault, AWS Secrets Manager).  
- Avoid hardcoding secrets in `.tf` files.  
- Use encrypted state storage.  
- Implement least privilege IAM roles.

15. **Terraform is destroying and recreating resources unnecessarily. Why?**  
Answer:  
Common causes:
- Resource attributes are computed and can't be predicted.  
- Changing immutable attributes (e.g., AMI ID on EC2 instance).  
- State file out of sync with actual infrastructure.  
- Provider bugs or version incompatibilities.

16. **How to rollback Terraform changes?**  
Answer:
- Use version control to revert to previous configuration.  
- Use `terraform state` commands to manipulate state.  
- Implement blue-green deployment patterns.  
- Use `terraform destroy` carefully (last resort).  
- Maintain state file backups.

---

### Category 5: Best Practices & Patterns

17. **What is Terraform backend configuration and why is it important?**  
Answer:  
Backend defines where state is stored and how operations are executed.

Importance:
- State persistence and sharing.  
- State locking for collaboration.  
- Remote operations for CI/CD.  
- Security and access control.

18. **How do you structure large Terraform projects?**  
Answer:
- **Monolithic**: Single configuration for all environments.  
- **Environment-based**: Separate configurations per environment.  
- **Module-based**: Reusable modules with environment-specific configurations.  
- **Hybrid**: Combination of above approaches.

19. **What is Terraform Cloud/Enterprise and its benefits?**  
Answer:  
Managed service for Terraform operations.

Benefits:
- Remote state storage.  
- Collaborative workspace management.  
- Policy as Code (Sentinel/OPA).  
- Private module registry.  
- Run triggers and notifications.

20. **How do you implement dependency management in Terraform?**  
Answer:
- **Implicit dependencies**: Terraform automatically detects through references.  
- **Explicit dependencies**: Use `depends_on` meta-argument.  
- **Module dependencies**: Through input/output variables.  
- **Data source dependencies**: Wait for resources to be created.

---

### Category 6: Advanced Scenarios

21. **What are Terraform data sources and when to use them?**  
Answer:  
Data sources fetch information from provider APIs.

Use Cases:
- Get AMI IDs dynamically.  
- Fetch VPC and subnet information.  
- Retrieve existing IAM roles.  
- Import existing resources data.

22. **How do you handle multiple environments in Terraform?**  
Answer:
- **Workspaces**: Same config, different states.  
- **Directory structure**: Separate directories per environment.  
- **Terragrunt**: Wrapper tool for DRY configurations.  
- **Git branches**: Different branches for environments.

23. **What is terraform taint and terraform untaint?**  
Answer:
- **taint**: Mark resource for recreation on next apply.  
- **untaint**: Remove taint mark.  
Use Case: Force resource recreation when troubleshooting.

24. **How do you manage Terraform provider versions?**  
Answer:
Example:
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

25. **What is Terraform state locking and why is it crucial?**  
Answer:  
Mechanism to prevent concurrent state modifications.

Importance:
- Prevents state corruption.  
- Enables team collaboration.  
- Ensures consistency in CI/CD pipelines.  
- Prevents resource conflicts.

---

### Category 7: Real-world Implementation

26. **How do you implement CI/CD with Terraform?**  
Answer:
- Store code in version control (Git).  
- Use remote state backend.  
- Implement automated testing (`terraform validate`, `plan`).  
- Use approval gates for production.  
- Implement policy checks (Sentinel/OPA).  
- Use Terraform Cloud/Enterprise for remote runs.

27. **What are Terraform dynamic blocks?**  
Answer:  
Generate multiple nested blocks dynamically.

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

28. **How do you handle conditional resource creation?**  
Answer:  
Use `count` or `for_each` meta-arguments.

Example:
```hcl
resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0

  ami           = var.ami
  instance_type = var.instance_type
}
```

29. **What is Terraform graph and how is it useful?**  
Answer:  
Visual representation of resource dependencies.

Use Cases:
- Understand complex dependency chains.  
- Troubleshoot creation/destruction order.  
- Optimize configuration structure.  
- Documentation of infrastructure relationships.

30. **How do you implement disaster recovery with Terraform?**  
Answer:
- Regular state file backups.  
- Version control for all configurations.  
- Modular design for easy recreation.  
- Documentation of manual steps (if any).  
- Regular `terraform plan` to detect drift.  
- Automated testing of configurations.

