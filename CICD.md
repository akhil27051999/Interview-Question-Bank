# DevOps Interview Preparation Guide

## Table of Contents
- [CI/CD](#ci-cd)
  - [Category 1: CI/CD Fundamentals](#category-1-ci-cd-fundamentals)
  - [Category 2: CI/CD Tools & Platforms](#category-2-ci-cd-tools--platforms)
  - [Category 3: Build & Test Automation](#category-3-build--test-automation)
  - [Category 4: Deployment Strategies](#category-4-deployment-strategies)
  - [Category 5: Security in CI/CD](#category-5-security-in-ci-cd)
  - [Category 6: Monitoring & Optimization](#category-6-monitoring--optimization)
  - [Category 7: Real-world Scenarios & Troubleshooting](#category-7-real-world-scenarios--troubleshooting)

---

## CI/CD

### Category 1: CI/CD Fundamentals

1. **What is CI/CD and explain the pipeline stages?**  
Answer:  
CI/CD stands for Continuous Integration and Continuous Delivery/Deployment.

Pipeline Stages:
- **Source**: Code commit triggers pipeline  
- **Build**: Compile code and run unit tests  
- **Test**: Integration, functional, performance tests  
- **Deploy**: Release to various environments  
- **Verify**: Post-deployment validation  
- **Monitor**: Ongoing performance monitoring

2. **Difference between Continuous Delivery vs Continuous Deployment?**  
Answer:  
- **Continuous Delivery**: Automated up to production, manual approval for release  
- **Continuous Deployment**: Fully automated including production release

Choice: Delivery for regulated environments, Deployment for rapid iteration

3. **What are the benefits of implementing CI/CD?**  
Answer:
- Faster time to market  
- Higher code quality  
- Reduced manual errors  
- Faster bug detection  
- Improved team collaboration  
- Consistent deployment process

4. **Explain blue-green deployment strategy.**  
Answer:  
Maintain two identical environments (blue - current, green - new).

Process:
- Deploy new version to green environment  
- Test green environment thoroughly  
- Switch traffic from blue to green  
- Blue becomes standby for rollback

Benefits: Zero downtime, easy rollback

---

### Category 2: CI/CD Tools & Platforms

5. **Compare Jenkins, GitLab CI, and GitHub Actions.**  
Answer:
- **Jenkins**: Open source, highly customizable, plugin ecosystem  
- **GitLab CI**: Integrated with GitLab, YAML-based, container-native  
- **GitHub Actions**: Integrated with GitHub, marketplace actions, YAML-based

Choice: Jenkins for complex custom needs, GitLab/GitHub for integrated solutions

6. **What is Jenkins Pipeline and how does it work?**  
Answer:  
Plugin suite supporting continuous delivery pipelines.

Types:
- **Declarative Pipeline**: Simplified syntax, ideal for most use cases  
- **Scripted Pipeline**: Groovy-based, more flexible and powerful

Example Declarative Pipeline:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

7. **Explain GitLab CI/CD configuration.**  
Answer:  
Configured via `.gitlab-ci.yml` file in repository.

Key Sections:
- **stages**: Define pipeline stages  
- **jobs**: Define individual tasks  
- **image**: Container image to use  
- **before_script**: Commands run before each job  
- **cache**: Files to cache between jobs

8. **What are GitHub Actions and key components?**  
Answer:  
Automation platform integrated with GitHub.

Components:
- **Workflows**: Automated processes defined in YAML  
- **Events**: Triggers for workflows (push, PR, schedule)  
- **Jobs**: Set of steps executed on same runner  
- **Actions**: Reusable units of code  
- **Runners**: Servers that run workflows

---

### Category 3: Build & Test Automation

9. **How do you manage dependencies in CI/CD?**  
Answer:
- Package Managers: Maven, npm, pip, NuGet  
- Container Images: Docker with multi-stage builds  
- Caching: Cache dependencies between builds  
- Private Registries: Nexus, Artifactory, ECR  
- Version Pinning: Specific versions for reproducibility

10. **What are different testing strategies in CI/CD?**  
Answer:
- Unit Tests: Fast, isolated, run on every build  
- Integration Tests: Verify component interactions  
- End-to-End Tests: Full system validation  
- Performance Tests: Load, stress, scalability testing  
- Security Tests: SAST, DAST, dependency scanning

11. **Explain code quality gates in CI/CD.**  
Answer:  
Quality thresholds that must be met before progression.

Common Gates:
- Test coverage percentage  
- Code complexity metrics  
- Security vulnerability count  
- Performance benchmarks  
- Linting rules compliance

12. **How to handle database migrations in CI/CD?**  
Answer:
- Versioned Migrations: Sequential SQL scripts  
- ORM Tools: Entity Framework, Liquibase, Flyway  
- Rollback Strategy: Backward-compatible changes  
- Environment-specific: Different strategies per environment  
- Blue-Green: Migrate before traffic switch

---

### Category 4: Deployment Strategies

13. **Compare rolling, blue-green, and canary deployments.**  
Answer:
- **Rolling**: Gradual replacement, mixed versions temporarily  
- **Blue-Green**: Instant switch, zero downtime, easy rollback  
- **Canary**: Gradual traffic shift, real-user testing

Choice: Rolling for simple apps, Blue-green for critical apps, Canary for risk reduction

14. **What is feature flagging and how is it used?**  
Answer:  
Technique to enable/disable features without code deployment.

Use Cases:
- Gradual feature rollouts  
- A/B testing  
- Kill switches for problematic features  
- Environment-specific configurations

15. **Explain infrastructure deployment strategies.**  
Answer:
- **Immutable Infrastructure**: Replace entire infrastructure, don't modify  
- **Mutable Infrastructure**: Update existing infrastructure in-place  
- **GitOps**: Declarative infrastructure as code with Git as source of truth

Choice: Immutable for consistency, Mutable for speed, GitOps for auditability

---

### Category 5: Security in CI/CD

16. **What is DevSecOps and key practices?**  
Answer:  
Integrating security practices into DevOps processes.

Key Practices:
- Security scanning in pipeline (SAST, DAST)  
- Dependency vulnerability scanning  
- Infrastructure security scanning  
- Secrets management  
- Compliance as code

17. **How to manage secrets in CI/CD pipelines?**  
Answer:
- Avoid: Hardcoded secrets in code  
- Use: Secret management tools (HashiCorp Vault, AWS Secrets Manager)  
- Pipeline Secrets: Use built-in secret storage (GitHub Secrets, GitLab CI Variables)  
- Temporary Credentials: Use OIDC or instance profiles  
- Rotation: Automatic secret rotation

18. **What is SAST and DAST in security testing?**  
Answer:
- **SAST (Static Application Security Testing)**: Analyze source code for vulnerabilities  
- **DAST (Dynamic Application Security Testing)**: Test running application for vulnerabilities

Use Both: SAST early in pipeline, DAST in later stages

---

### Category 6: Monitoring & Optimization

19. **How to monitor CI/CD pipeline performance?**  
Answer:
- Metrics: Build duration, success rates, queue times  
- Logging: Detailed execution logs  
- Alerting: Failures, performance degradation  
- Dashboards: Visual pipeline health monitoring  
- Tracing: End-to-end request tracing

20. **What are common CI/CD pipeline bottlenecks?**  
Answer:
- Compute Resources: Insufficient build agents  
- Network: Slow dependency downloads  
- Tests: Long-running test suites  
- Approvals: Manual approval delays  
- Deployment: Environment availability

21. **How to optimize CI/CD pipeline speed?**  
Answer:
- Parallelize independent tasks  
- Implement effective caching  
- Use faster build agents  
- Optimize test suites (test selection, parallel execution)  
- Use container layers effectively  
- Implement incremental builds

---

### Category 7: Real-world Scenarios & Troubleshooting

22. **Pipeline fails randomly due to flaky tests. How to handle?**  
Answer:
- Identify and fix flaky tests  
- Implement test retry mechanisms  
- Quarantine problematic tests  
- Use test stability metrics  
- Implement test parallelization

23. **Build times are increasing significantly. How to optimize?**  
Answer:
- Analyze build duration metrics  
- Implement dependency caching  
- Use build cache (Docker layer cache, Gradle cache)  
- Parallelize build steps  
- Optimize Dockerfile with multi-stage builds  
- Use faster build machines

24. **Deployment fails in production. How to perform rollback?**  
Answer:
- Automated rollback triggers  
- Blue-green deployment switch  
- Database rollback procedures  
- Feature flag disable  
- Monitor rollback success  
- Post-mortem analysis

25. **How to handle environment-specific configurations?**  
Answer:
- Configuration Management: Ansible, Chef, Puppet  
- Template-based: Helm charts, CloudFormation templates  
- Environment Variables: Different values per environment  
- Configuration Services: Spring Cloud Config, AWS AppConfig  
- Secrets Management: External secret storage

26. **Multiple teams contributing to same pipeline. How to manage?**  
Answer:
- Pipeline as code in repository  
- Modular pipeline design  
- Approval gates for cross-team changes  
- Shared pipeline libraries  
- Environment isolation  
- Clear ownership and responsibilities

27. **How to implement CI/CD for microservices architecture?**  
Answer:
- Independent Pipelines: Separate pipeline per service  
- Shared Libraries: Common pipeline components  
- Dependency Management: Versioned API contracts  
- Integration Testing: Service mesh testing  
- Deployment Coordination: Feature flags, API versioning

28. **What is GitOps and how does it work?**  
Answer:  
Operational framework using Git as single source of truth.

Principles:
- Declarative infrastructure and applications  
- Version controlled system state  
- Automated synchronization  
- Software agents ensuring correctness

Rollback and audit through Git

29. **How to secure the CI/CD pipeline itself?**  
Answer:
- Access Control: Least privilege principles  
- Pipeline Security: Scan pipeline definitions  
- Build Environment: Isolated, ephemeral build agents  
- Artifact Integrity: Signed artifacts, provenance  
- Audit Logging: Comprehensive activity logs

30. **Explain disaster recovery for CI/CD systems.**  
Answer:
- Backup: Pipeline configurations, credentials, artifacts  
- Recovery: Automated environment recreation  
- Multi-region: Cross-region pipeline deployment  
- Documentation: Manual procedures for critical scenarios  
- Testing: Regular disaster recovery drills
