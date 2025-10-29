# DevOps Interview Preparation Guide

## Table of Contents
- [Monitoring](#monitoring)
  - [Category 1: Monitoring Fundamentals](#category-1-monitoring-fundamentals)
  - [Category 2: Metrics & Alerting](#category-2-metrics--alerting)
  - [Category 3: Logging Management](#category-3-logging-management)
  - [Category 4: Distributed Tracing](#category-4-distributed-tracing)
  - [Category 5: Monitoring Tools & Platforms](#category-5-monitoring-tools--platforms)
  - [Category 6: Application Performance Monitoring (APM)](#category-6-application-performance-monitoring-apm)
  - [Category 7: Container & Kubernetes Monitoring](#category-7-container--kubernetes-monitoring)
  - [Category 8: Cloud Monitoring](#category-8-cloud-monitoring)
  - [Category 9: Real-time Troubleshooting](#category-9-real-time-troubleshooting)

---

## Monitoring

### Category 1: Monitoring Fundamentals

1. **What is the difference between monitoring and observability?**  
Answer:  
Monitoring: Proactive collection and analysis of predefined metrics.  
Observability: Ability to understand system internals through external outputs.  
Monitoring tells you when something is wrong; observability helps you understand why.

2. **Explain the Three Pillars of Observability.**  
Answer:  
- Metrics: Numerical measurements over time (CPU usage, error rates).  
- Logs: Timestamped events with structured context.  
- Traces: End-to-end journey of requests through distributed systems.  
- Fourth Pillar (emerging): Profiles — resource usage by code.

3. **What are SLIs, SLOs, and SLAs?**  
Answer:  
- SLI (Service Level Indicator): Measurable aspect of service (availability, latency).  
- SLO (Service Level Objective): Target value for an SLI.  
- SLA (Service Level Agreement): Contractual agreement with consequences for missing SLOs.  
Example: SLI = uptime%, SLO = 99.9%, SLA = service credits if <99.9%.

4. **Compare white-box vs black-box monitoring.**  
Answer:  
- White-box: Monitoring from inside the system (application metrics, logs).  
- Black-box: Monitoring from outside perspective (synthetic transactions, uptime checks).  
Use both: White-box for deep insights, black-box for user perspective.

---

### Category 2: Metrics & Alerting

5. **What are the Four Golden Signals of monitoring?**  
Answer:  
- Latency: Time to serve requests.  
- Traffic: Demand on the system (requests/second).  
- Errors: Rate of failed requests.  
- Saturation: How "full" the service is (utilization).

6. **Explain metric types: Counter, Gauge, Histogram.**  
Answer:  
- Counter: Monotonically increasing value (total requests, errors).  
- Gauge: Value that can go up and down (memory usage, active connections).  
- Histogram: Samples observations into configurable buckets (request duration).  
- Summary: Similar to histogram but calculates quantiles client-side.

7. **What makes a good alert?**  
Answer:  
- Actionable: Requires human or automated intervention.  
- Specific: Clear what's wrong and potential impact.  
- Relevant: Not noisy or redundant.  
- Timely: Fires when action can be taken.  
- Documented: Includes a runbook for resolution.

8. **How to avoid alert fatigue?**  
Answer:  
- Implement alert hierarchy and routing (PagerDuty, Opsgenie).  
- Use meaningful thresholds and grouping.  
- Suppress or deduplicate related alerts.  
- Regularly review and cleanup alerts.  
- Route based on time/severity and use ML for anomaly detection where appropriate.

---

### Category 3: Logging Management

9. **What is structured logging and why is it important?**  
Answer:  
Logging in a structured format (e.g., JSON) rather than plain text.

Benefits:
- Easier parsing and querying.  
- Better integration with log analysis tools.  
- Consistent field extraction.  
- Enhanced correlation capabilities.

Example:
```json
{
  "timestamp": "2023-01-15T10:30:00Z",
  "level": "ERROR",
  "message": "Failed to process order",
  "order_id": "12345",
  "user_id": "67890",
  "error": "Insufficient inventory"
}
```

10. **Compare centralized vs distributed logging.**  
Answer:  
- Centralized: All logs sent to a single system (ELK, Splunk) for correlation and analysis.  
- Distributed: Logs remain with services and are queried when needed (can be more scalable).  
- Hybrid Approach: Recent logs centralized for fast troubleshooting, historical logs archived/distributed for scale.  
Choice depends on scale, cost, and correlation needs.

11. **What is log rotation and why is it necessary?**  
Answer:  
Process of limiting log file size and age by rotating, compressing, and archiving logs.

Reasons:
- Prevent disk space exhaustion.  
- Improve log management and ingestion performance.  
- Facilitate log archival and retention policies.

---

### Category 4: Distributed Tracing

12. **What is distributed tracing and how does it work?**  
Answer:  
Technique for tracking requests across multiple services.

Key Concepts:
- Trace: Complete request journey.  
- Span: Individual operation within a trace.  
- Context Propagation: Passing trace information between services.  
- Sampling: Selecting which traces to record to control volume.

13. **Explain OpenTelemetry and its components.**  
Answer:  
Vendor-neutral observability framework.

Components:
- API: Language-specific instrumentation APIs.  
- SDK: Implementation of the API and configuration.  
- Collector: Receives, processes, and exports telemetry data.  
- Instrumentation: Automatic and manual code instrumentation.

14. **When should you use sampling in tracing?**  
Answer:  
- Always sample for debugging specific issues or low-traffic services.  
- Probabilistic sampling for general monitoring in moderate traffic.  
- Rate limiting for very high-traffic services.  
- Adaptive sampling based on error rate or other conditions to capture important traces.

---

### Category 5: Monitoring Tools & Platforms

15. **Compare Prometheus and Datadog.**  
Answer:  
- Prometheus: Open source, pull-based, powerful query language (PromQL), self-hosted.  
- Datadog: SaaS, mostly push-based, extensive integrations and enterprise features.  
Choice: Prometheus for cost control and customization; Datadog for ease of use and enterprise features.

16. **What is Grafana and how is it used?**  
Answer:  
Open source visualization and analytics platform.

Use Cases:
- Dashboard creation for metrics.  
- Alert configuration and notification.  
- Aggregating multiple data sources (Prometheus, Elasticsearch, etc.).  
- Team collaboration and sharing.

17. **Explain ELK Stack components.**  
Answer:  
- Elasticsearch: Distributed search and analytics engine.  
- Logstash: Server-side data processing pipeline.  
- Kibana: Visualization and management interface.  
- Beats: Lightweight shippers (Filebeat, Metricbeat) for collecting logs/metrics.

---

### Category 6: Application Performance Monitoring (APM)

18. **What is APM and what does it monitor?**  
Answer:  
Application Performance Monitoring focuses on application-level metrics and behavior.

Monitors:
- Transaction traces.  
- Database query performance.  
- External service calls.  
- Application errors and exceptions.  
- Code-level performance.

19. **Compare infrastructure monitoring vs APM.**  
Answer:  
- Infrastructure Monitoring: Tracks CPU, memory, disk, network metrics.  
- APM: Focuses on application transactions, business metrics, and user experience.  
Need both: Infrastructure for resource issues; APM for application-level bottlenecks.

20. **What are real-user monitoring (RUM) and synthetic monitoring?**  
Answer:  
- RUM: Monitoring actual user interactions with the application (client-side).  
- Synthetic: Automated scripts simulating user behavior to test availability and performance.  
Use both: RUM for real user experience; Synthetic for predictable checks and SLAs.

---

### Category 7: Container & Kubernetes Monitoring

21. **How to monitor Kubernetes clusters effectively?**  
Answer:  
- Cluster-level: Node resources, pod status, API server metrics.  
- Application-level: Application metrics and business logic metrics.  
- Network: Service mesh metrics and network policy metrics.  
- Storage: Persistent volume usage and IOPS.  
Tools: Prometheus, Grafana, Datadog, New Relic, etc.

22. **What is cAdvisor and how does it work?**  
Answer:  
cAdvisor (Container Advisor) is an open source tool from Google.

Features:
- Collects container resource usage and performance characteristics.  
- Exposes metrics via REST API.  
- Integrates with kubelet and provides historical usage stats.

23. **Explain Kubernetes monitoring best practices.**  
Answer:  
- Monitor at multiple levels (cluster, node, pod, container).  
- Implement horizontal pod autoscaling based on metrics.  
- Use liveness and readiness probes effectively.  
- Monitor etcd performance.  
- Implement distributed tracing for microservices.

---

### Category 8: Cloud Monitoring

24. **How to monitor AWS resources effectively?**  
Answer:  
- Use CloudWatch for metrics and logs.  
- Use CloudTrail for API activity logging and auditing.  
- Use X-Ray for distributed tracing.  
- Integrate third-party tools (Datadog, New Relic).  
- Emit custom application/business metrics to CloudWatch.

25. **What is CloudWatch and its key features?**  
Answer:  
AWS monitoring and observability service.

Features:
- Metrics collection and visualization.  
- Log aggregation and analysis (CloudWatch Logs).  
- Alarm creation and notification.  
- Dashboard creation and automatic scaling triggers.

---

### Category 9: Real-time Troubleshooting

26. **Service is slow but CPU/memory usage is normal. How to troubleshoot?**  
Answer:  
- Check application logs for errors and latency spikes.  
- Analyze database query performance and slow queries.  
- Check external API response times and network latency.  
- Examine garbage collection patterns and thread contention.  
- Use distributed tracing to identify bottlenecks in request flow.

27. **How to identify memory leaks in production?**  
Answer:  
- Monitor memory usage trends over time across instances.  
- Analyze garbage collection logs and patterns.  
- Capture heap dumps for detailed analysis.  
- Compare memory usage across versions and instances.  
- Use canary deployments to isolate issues.

28. **Database performance degradation. How to investigate?**  
Answer:  
- Check database connection pool usage and saturation.  
- Analyze slow query logs and execution plans.  
- Monitor locks, deadlocks, and transaction contention.  
- Check index usage and fragmentation.  
- Review application query patterns and caching.  
- Monitor disk I/O and cache hit ratios.

29. **How to monitor microservices communication?**  
Answer:  
- Implement distributed tracing to follow requests across services.  
- Monitor service mesh metrics (Istio, Linkerd) if used.  
- Track inter-service latency and error rates.  
- Monitor circuit breaker and retry metrics.  
- Use correlation IDs to trace requests in logs.

30. **What is chaos engineering and how does it relate to monitoring?**  
Answer:  
Chaos engineering is the practice of experimenting on systems to build confidence in their resilience.

Relation to Monitoring:
- Tests monitoring system effectiveness and alerting.  
- Validates alert thresholds and runbooks.  
- Identifies monitoring gaps and improves incident response.  
- Helps ensure observability under failure conditions.
