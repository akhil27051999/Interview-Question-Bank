# DevOps Interview Preparation Guide — CI/CD (Reformatted)

This file contains the CI/CD section of the DevOps Interview Preparation Guide, reformatted for clarity and interview use.  
Each question is a numbered heading (Q#). Answers are placed under a bold **Answer** subheading. Code examples and snippets are in fenced code blocks. Use this as a study reference, cheat-sheet, or to include in your main README.

---

## Table of Contents
- [Category 1: CI/CD Fundamentals](#category-1-ci-cd-fundamentals)
- [Category 2: CI/CD Tools & Platforms](#category-2-ci-cd-tools--platforms)
- [Category 3: Build & Test Automation](#category-3-build--test-automation)
- [Category 4: Deployment Strategies](#category-4-deployment-strategies)
- [Category 5: Security in CI/CD](#category-5-security-in-ci-cd)
- [Category 6: Monitoring & Optimization](#category-6-monitoring--optimization)
- [Category 7: Real-world Scenarios & Troubleshooting](#category-7-real-world-scenarios--troubleshooting)

---

## CI/CD

---

### Category 1: CI/CD Fundamentals

#### Q1 — What is CI/CD and explain the pipeline stages?
**Answer**

CI/CD stands for Continuous Integration and Continuous Delivery/Deployment.

Typical pipeline stages:
- **Source:** Code commit triggers pipeline.  
- **Build:** Compile code, build artifacts, run unit tests.  
- **Test:** Integration, functional, and performance tests.  
- **Deploy:** Release to staging/production environments.  
- **Verify:** Post-deployment validation (smoke tests).  
- **Monitor:** Ongoing performance and health monitoring.

---

#### Q2 — Difference between Continuous Delivery vs Continuous Deployment?
**Answer**

- **Continuous Delivery:** Pipeline is automated up to production; a manual approval step releases to production.  
- **Continuous Deployment:** Every passing change is automatically deployed to production (no manual gate).

Choose Delivery for regulated or high-risk environments; Deployment for rapid iteration with strong automated tests.

---

#### Q3 — What are the benefits of implementing CI/CD?
**Answer**

- Faster time to market  
- Higher code quality and earlier bug detection  
- Reduced manual errors and repeatability  
- Improved developer productivity and collaboration  
- Consistent, auditable deployment process

---

#### Q4 — Explain blue-green deployment strategy.
**Answer**

- Maintain two identical environments: **Blue** (current) and **Green** (new).  
- Deploy the new version to the green environment and test it thoroughly.  
- Switch traffic from blue to green (DNS or load balancer).  
- Keep blue as rollback standby.

Benefits: Zero downtime and quick rollback.

---

### Category 2: CI/CD Tools & Platforms

#### Q5 — Compare Jenkins, GitLab CI, and GitHub Actions.
**Answer**

- **Jenkins:** Open-source, extensible via plugins, supports complex/custom workflows.  
- **GitLab CI:** Integrated with GitLab, YAML pipelines, container-native runners, built-in features.  
- **GitHub Actions:** Integrated with GitHub, marketplace of actions, simple YAML workflows.

Pick Jenkins for complex custom needs, GitLab/GitHub for integrated developer workflows.

---

#### Q6 — What is Jenkins Pipeline and how does it work?
**Answer**

Jenkins Pipeline is a suite of plugins that supports building continuous delivery pipelines as code.

- **Declarative Pipeline:** Simpler, structured syntax for common use.  
- **Scripted Pipeline:** Groovy-based, more flexible and powerful.

Example (Declarative):
```groovy
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'mvn compile' } }
    stage('Test')  { steps { sh 'mvn test'    } }
  }
}
```

---

#### Q7 — Explain GitLab CI/CD configuration.
**Answer**

Configured via `.gitlab-ci.yml` in the repo.

Key sections:
- `stages`: Define pipeline stages.  
- `jobs`: Tasks assigned to stages.  
- `image`: Container image to run job.  
- `before_script`: Commands run before jobs.  
- `cache`: Cache files between jobs to speed builds.

---

#### Q8 — What are GitHub Actions and key components?
**Answer**

GitHub Actions is an automation platform integrated with GitHub.

Components:
- **Workflows:** Defined in YAML under `.github/workflows/`.  
- **Events:** Triggers (push, PR, schedule).  
- **Jobs:** Groups of steps run on the same runner.  
- **Actions:** Reusable tasks from the marketplace.  
- **Runners:** Machines that execute jobs (hosted or self-hosted).

---

### Category 3: Build & Test Automation

#### Q9 — How do you manage dependencies in CI/CD?
**Answer**

- Use package managers (Maven, npm, pip, NuGet).  
- Build container images (Docker) with multi-stage builds.  
- Cache dependencies between builds (CI cache, Docker layer cache).  
- Use private registries (Nexus, Artifactory, ECR) for reproducibility.  
- Pin dependency versions for deterministic builds.

---

#### Q10 — What are different testing strategies in CI/CD?
**Answer**

- **Unit Tests:** Fast, run on every commit.  
- **Integration Tests:** Verify component interactions.  
- **End-to-End (E2E) Tests:** Validate the full user flow.  
- **Performance Tests:** Load, stress, and scalability testing.  
- **Security Tests:** SAST, DAST, dependency vulnerability scans.

Test pyramid: Many unit tests, fewer integration tests, even fewer E2E tests.

---

#### Q11 — Explain code quality gates in CI/CD.
**Answer**

Quality gates enforce thresholds before allowing progression:

Common gates:
- Minimum test coverage percentage  
- No critical security vulnerabilities (SAST)  
- Linting/style compliance  
- Code complexity/duplication thresholds  
- Performance or benchmark checks

Fail the pipeline if gates are not met.

---

#### Q12 — How to handle database migrations in CI/CD?
**Answer**

- Use versioned migrations (Flyway, Liquibase, Rails/Rake migrations) stored in source control.  
- Apply migrations as part of the deployment pipeline before switching traffic (or use backwards-compatible changes).  
- Have rollback or compensating scripts where possible.  
- Use blue-green/canary strategies for schema changes that require coordination.  
- Test migrations in staging environments with production-like data.

---

### Category 4: Deployment Strategies

#### Q13 — Compare rolling, blue-green, and canary deployments.
**Answer**

- **Rolling:** Replace instances incrementally (some overlap of versions).  
- **Blue-Green:** Deploy to green, switch traffic instantly; blue retained for rollback.  
- **Canary:** Shift a small % of traffic to new version, monitor, then increase %.

Choose based on risk tolerance, complexity, and ability to monitor.

---

#### Q14 — What is feature flagging and how is it used?
**Answer**

Feature flags (feature toggles) allow enabling/disabling features at runtime without redeploying.

Uses:
- Gradual rollouts and A/B testing.  
- Fast rollback by toggling off.  
- Decouple deploy from release.  
- Control per-user or per-environment exposure.

Tools: LaunchDarkly, Unleash, Flagr, homegrown toggles.

---

#### Q15 — Explain infrastructure deployment strategies.
**Answer**

- **Immutable Infrastructure:** Replace instances/containers rather than patching in place (reduces configuration drift).  
- **Mutable Infrastructure:** Update live machines (faster but riskier).  
- **GitOps:** Declarative infra in Git with automated reconciliation (Flux, Argo CD).

Immutable is preferred for reproducibility; GitOps adds auditability and automation.

---

### Category 5: Security in CI/CD

#### Q16 — What is DevSecOps and key practices?
**Answer**

DevSecOps integrates security into the DevOps lifecycle.

Key practices:
- Shift-left security (SAST in pre-commit/CI).  
- Dependency vulnerability scanning.  
- Secret detection and management.  
- Infrastructure scanning (IaC security).  
- Automated compliance checks and policy-as-code.

---

#### Q17 — How to manage secrets in CI/CD pipelines?
**Answer**

- Never hardcode secrets in code or repo.  
- Use secret managers (HashiCorp Vault, AWS Secrets Manager).  
- Use CI built-in secrets stores (GitHub Secrets, GitLab CI variables) with restricted access.  
- Prefer short-lived credentials (OIDC, cloud provider roles) over long-lived keys.  
- Rotate secrets routinely.

---

#### Q18 — What is SAST and DAST in security testing?
**Answer**

- **SAST (Static Application Security Testing):** Analyze source code for vulnerabilities at rest (early in pipeline).  
- **DAST (Dynamic Application Security Testing):** Test running applications for vulnerabilities (runtime, later stage).

Use both: SAST early for code issues, DAST on deployed or staging environments.

---

### Category 6: Monitoring & Optimization

#### Q19 — How to monitor CI/CD pipeline performance?
**Answer**

Track metrics and telemetry:
- Pipeline duration and stage durations.  
- Build success/failure rates and flakiness.  
- Queue times and runner utilization.  
- Artifact size and storage metrics.  
- Create dashboards, alerts, and trend analysis.

---

#### Q20 — What are common CI/CD pipeline bottlenecks?
**Answer**

- Insufficient build agents or runners.  
- Slow network or dependency downloads.  
- Long-running or flaky test suites.  
- Manual approval gates causing delays.  
- Slow deploy steps or environment provisioning.

---

#### Q21 — How to optimize CI/CD pipeline speed?
**Answer**

- Parallelize independent steps and tests.  
- Cache dependencies and build artifacts.  
- Use faster or specialized runners (e.g., containers vs VMs).  
- Split monolith builds into smaller modules.  
- Run only impacted tests (test selection).  
- Use multi-stage Docker builds and layer caching.

---

### Category 7: Real-world Scenarios & Troubleshooting

#### Q22 — Pipeline fails randomly due to flaky tests. How to handle?
**Answer**

- Identify flaky tests (track failure rates).  
- Quarantine or mark flaky tests until fixed.  
- Implement retries with backoff for non-deterministic failures.  
- Improve test isolation and determinism (mock external dependencies).  
- Invest in test reliability (timeouts, cleanup).

---

#### Q23 — Build times are increasing significantly. How to optimize?
**Answer**

- Analyze where time is spent (CI metrics).  
- Implement dependency and layer caching (Docker/Gradle/NPM).  
- Parallelize build/test steps.  
- Use incremental builds and artifact reuse.  
- Upgrade or scale build agents.

---

#### Q24 — Deployment fails in production. How to perform rollback?
**Answer**

Rollback approaches:
- **Automated rollback:** Triggered by failed health checks or alarms.  
- **Blue-Green:** Switch traffic back to previous environment.  
- **Feature Flag:** Disable new feature(s) instantly.  
- **Database:** Apply reverse migration or use backward-compatible schema designs.  
- After rollback, perform a postmortem and harden pipeline to prevent recurrence.

---

#### Q25 — How to handle environment-specific configurations?
**Answer**

- Use environment-specific configuration files or templates (e.g., Helm values, CloudFormation parameters).  
- Use environment variables and secret stores per environment.  
- Utilize configuration services (AWS AppConfig, Spring Cloud Config).  
- Manage environment differences via overlays or separate parameterized deployments.

---

#### Q26 — Multiple teams contributing to same pipeline. How to manage?
**Answer**

- Treat pipeline as code with clear ownership and PR review processes.  
- Create modular/reusable pipeline components or libraries.  
- Enforce policies and approvals for cross-team changes.  
- Use role-based access and environment isolation.  
- Maintain documentation and versioning for shared pipeline modules.

---

#### Q27 — How to implement CI/CD for microservices architecture?
**Answer**

- Independent pipelines per service for autonomy.  
- Shared pipeline templates or libraries for consistency.  
- Contract testing and consumer-driven contracts for integration.  
- Use service discovery and versioned APIs.  
- Coordinate releases with feature flags, canaries, and semantic versioning.

---

#### Q28 — What is GitOps and how does it work?
**Answer**

GitOps uses Git as the single source of truth for declarative infrastructure and applications.

Principles:
- All desired system state is in Git.  
- Automated agents (controllers) reconcile Git state to the cluster.  
- Changes are pull-request-driven, auditable, and reversible.

Tools: Flux, Argo CD.

---

#### Q29 — How to secure the CI/CD pipeline itself?
**Answer**

- Enforce least privilege for pipeline credentials and secrets.  
- Use ephemeral build agents and isolated environments.  
- Scan pipeline definitions for insecure steps.  
- Sign and verify artifacts; ensure provenance.  
- Audit pipeline activity and access logs.

---

#### Q30 — Explain disaster recovery for CI/CD systems.
**Answer**

DR considerations:
- Backup pipeline definitions, configuration, and secrets (securely).  
- Store artifacts in durable remote registries (artifact repositories).  
- Automate environment bootstrapping (IaC) to recreate CI/CD infrastructure.  
- Use multi-region or multi-account redundancy for critical services.  
- Regularly test restore and failover procedures.

