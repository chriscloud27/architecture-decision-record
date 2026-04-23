# Decision record template for Well-Architected Framework

This template structures architecture decisions around the six pillars of the Well-Architected Framework, enabling consistent evaluation across AWS, Azure, and GCP.

References:
* [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
* [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
* [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)

---

# [Title: present-tense imperative phrase describing the decision]

**Status**: [proposed | accepted | rejected | deprecated | superseded by ADR-XXXX]

**Deciders**: [names or roles of decision-makers]

**Date**: [YYYY-MM-DD]

**Tags**: [e.g., cloud, ai, security, cost, sustainability]

---

## Context and Problem Statement

[Describe the architectural challenge, business driver, or opportunity. What is forcing this decision now? Include relevant constraints such as compliance requirements, scale targets, or strategic goals.]

## Decision Drivers

* [Driver 1: e.g., regulatory compliance requirement]
* [Driver 2: e.g., target SLA of 99.9% availability]
* [Driver 3: e.g., cost reduction target]
* [Driver 4: e.g., AI-readiness or AI-native platform strategy]

## Considered Options

* [Option A]
* [Option B]
* [Option C]

## Decision Outcome

**Chosen option**: [Option X], because [brief justification linking back to decision drivers].

---

## Well-Architected Framework Assessment

### Operational Excellence

> *Run and monitor systems to deliver business value, and continually improve supporting processes and procedures.*

**Impact of this decision**:

* Monitoring and observability: [How does this decision affect telemetry, alerting, and dashboards?]
* Automation and IaC: [Does this decision enable or require infrastructure-as-code, CI/CD pipelines, or runbook automation?]
* Incident response: [How does this affect runbooks, on-call burden, or mean time to recovery (MTTR)?]
* Team cognitive load: [What operational skills and tooling are required?]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

### Security

> *Protect information, systems, and assets while delivering business value through risk assessments and mitigation strategies.*

**Impact of this decision**:

* Identity and access management: [IAM roles, least-privilege, service accounts, federation]
* Data protection: [Encryption at rest and in transit, key management, data classification]
* Threat detection: [Logging, SIEM integration, vulnerability scanning, intrusion detection]
* Compliance: [Relevant standards: ISO 27001, SOC 2, HIPAA, PCI-DSS, GDPR, FedRAMP, etc.]
* Network security: [VPC/VNET design, private endpoints, WAF, DDoS protection]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

### Reliability

> *Ensure a workload performs its intended function correctly and consistently when it is expected to.*

**Impact of this decision**:

* Availability targets: [SLA/SLO goals and how this decision supports them]
* Fault isolation: [Blast radius, availability zones, multi-region strategy]
* Recovery: [RTO/RPO targets, backup and restore, disaster recovery runbooks]
* Resilience patterns: [Circuit breakers, retries with backoff, bulkhead isolation]
* Dependency management: [Single points of failure, health checks, dependency maps]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

### Performance Efficiency

> *Use computing resources efficiently to meet system requirements, and maintain that efficiency as demand changes and technologies evolve.*

**Impact of this decision**:

* Scaling strategy: [Horizontal vs. vertical, auto-scaling triggers, scale-to-zero]
* Latency targets: [p50/p95/p99 latency budgets for critical paths]
* Resource utilization: [CPU, memory, GPU, storage efficiency baselines]
* Caching and data locality: [Cache tiers, CDN usage, data placement strategy]
* Technology selection: [Right service/runtime for the workload; serverless, containers, VMs, AI accelerators]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

### Cost Optimization

> *Avoid unnecessary costs by understanding spending over time, controlling fund allocation, and scaling to meet business needs without overspending.*

**Impact of this decision**:

* Pricing model: [On-demand, reserved/committed use, spot/preemptible, serverless]
* Total cost of ownership (TCO): [Licensing, operations, training, migration costs]
* FinOps practices: [Tagging strategy, cost allocation, budgets and alerts, rightsizing]
* Build vs. buy: [Make-or-buy analysis, managed services vs. self-hosted]
* Cost of inaction: [What does it cost *not* to make this decision?]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

### Sustainability

> *Minimize the environmental impact of running cloud workloads by reducing energy consumption and improving efficiency across all components.*

**Impact of this decision**:

* Carbon footprint: [Estimated CO₂ impact; use of low-carbon or renewable-energy regions]
* Resource efficiency: [Idle resource elimination, right-sizing, serverless/scale-to-zero]
* Data transfer minimization: [Reducing unnecessary cross-region or cross-AZ data movement]
* Hardware lifecycle: [Shared infrastructure preference over dedicated; avoiding e-waste]
* Sustainability tooling: [AWS Customer Carbon Footprint Tool, Azure Emissions Impact Dashboard, GCP Carbon Footprint]

**Alignment**: [High | Medium | Low] — [brief explanation]

---

## Trade-off Summary

| Pillar | Alignment | Notes |
|---|---|---|
| Operational Excellence | [High/Medium/Low] | |
| Security | [High/Medium/Low] | |
| Reliability | [High/Medium/Low] | |
| Performance Efficiency | [High/Medium/Low] | |
| Cost Optimization | [High/Medium/Low] | |
| Sustainability | [High/Medium/Low] | |

**Key trade-offs**: [Describe any tensions between pillars, e.g., "Higher reliability via multi-region increases cost and carbon footprint; mitigated by active-passive design."]

---

## Pros and Cons of the Options

### [Option A]

* Good, because [argument aligned with a WAF pillar]
* Good, because [argument aligned with a WAF pillar]
* Bad, because [argument; which pillar is compromised]

### [Option B]

* Good, because [argument aligned with a WAF pillar]
* Bad, because [argument; which pillar is compromised]
* Bad, because [argument; which pillar is compromised]

### [Option C]

* Good, because [argument aligned with a WAF pillar]
* Bad, because [argument; which pillar is compromised]

---

## Consequences

### Positive

* [e.g., Automated scaling reduces operational overhead]
* [e.g., Managed service eliminates patching burden]

### Negative

* [e.g., Vendor lock-in to a specific cloud provider service]
* [e.g., Higher upfront cost compared to self-hosted alternative]

### Neutral / Follow-up decisions required

* [e.g., A separate ADR is needed for the logging and alerting strategy]
* [e.g., Security review required before production deployment]

---

## Links and References

* [Link type: e.g., Supersedes] [ADR-XXXX](../XXXX-title/)
* [Link type: e.g., Related to] [ADR-YYYY](../YYYY-title/)
* [Cloud provider WAF review or Well-Architected Tool report URL]
* [Relevant architecture diagrams or runbooks]
