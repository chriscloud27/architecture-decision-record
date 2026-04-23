# Decision record template for WAF++

This template structures architecture decisions using the **WAF++** (Well-Architected Framework++) seven-pillar model.
WAF++ is an open-source, vendor-neutral community framework. See [waf2p.dev](https://waf2p.dev/) and [github.com/WAF2p/framework](https://github.com/WAF2p/framework).

---

# [Title: present-tense imperative phrase describing the decision]

**Status**: [proposed | accepted | rejected | deprecated | superseded by ADR-XXXX]

**Deciders**: [names or roles of decision-makers]

**Date**: [YYYY-MM-DD]

**Tags**: [e.g., cloud, platform, security, governance, data-sovereignty]

**WAF++ Review**: [not started | in progress | completed — link to review record]

---

## Context and Problem Statement

[Describe the architectural challenge, business driver, or opportunity. Include any regulatory, sovereignty, or compliance constraints that make this decision significant.]

## Decision Drivers

* [Driver 1: e.g., GDPR data residency requirement]
* [Driver 2: e.g., target SLA of 99.9% availability]
* [Driver 3: e.g., vendor exit strategy / portability requirement]
* [Driver 4: e.g., cost reduction target or FinOps mandate]

## Considered Options

* [Option A]
* [Option B]
* [Option C]

## Decision Outcome

**Chosen option**: [Option X], because [brief justification linking back to decision drivers].

**Evidence reference**: [Link to architecture review record, WAF++ assessment, or audit evidence]

---

## WAF++ Seven-Pillar Assessment

> Assess each pillar at one of three maturity levels:
> **Baseline** (minimum viable) · **Standardize** (repeatable/automated) · **Optimize** (measurable KPI-driven)

---

### Pillar 1 — Security

> *Controls, threat modelling, policy-as-code, secure defaults.*

**Impact of this decision**:

* Identity and access: [IAM model, least-privilege, federation, workload identity]
* Data protection: [Encryption at rest and in transit, key management, data classification]
* Threat detection and response: [Logging, SIEM, vulnerability scanning, runtime security]
* Network security: [Segmentation, private endpoints, firewall rules, zero-trust]
* Policy as code: [OPA/Rego, cloud policy engines, guardrails, drift detection]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to threat model, security control matrix, or penetration test summary]

---

### Pillar 2 — Cost Optimization

> *FinOps, cost transparency, budgets, guardrails, right-sizing.*

**Impact of this decision**:

* Pricing model: [On-demand, reserved/committed, spot/preemptible, serverless, pay-per-use]
* Total cost of ownership (TCO): [Licensing, operations, training, migration, exit costs]
* FinOps practices: [Tagging taxonomy, cost allocation, budget alerts, rightsizing cadence]
* Cost guardrails: [Automated spend controls, anomaly detection, per-team budgets]
* Build vs. buy: [Open-source vs. managed service trade-off analysis]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to cost model, FinOps dashboard, or TCO analysis]

---

### Pillar 3 — Performance Efficiency

> *Scalability, latency, load patterns, resource tuning.*

**Impact of this decision**:

* Scaling strategy: [Horizontal/vertical, auto-scaling, scale-to-zero, event-driven]
* Latency targets: [p50/p95/p99 budgets for critical paths]
* Resource utilization: [CPU, memory, GPU, I/O efficiency baselines]
* Load patterns: [Steady state vs. burst, batch vs. real-time, geographic distribution]
* Technology fit: [Correct runtime for the workload: serverless, containers, VMs, FPGAs]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to load test results, capacity model, or benchmark report]

---

### Pillar 4 — Reliability

> *SLOs, backups, failover, chaos testing, disaster recovery.*

**Impact of this decision**:

* Availability targets: [SLA/SLO definitions and measurement approach]
* Fault isolation: [Blast radius, availability zones, cell-based architecture]
* Recovery: [RTO/RPO targets, backup schedule, restore testing frequency]
* Resilience patterns: [Circuit breakers, retries with backoff, bulkhead isolation]
* Chaos engineering: [Failure injection practice, game days, dependency mapping]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to SLO report, DR runbook, or chaos engineering results]

---

### Pillar 5 — Operational Excellence

> *Developer experience, standards, paved road, self-service.*

**Impact of this decision**:

* Paved road / golden path: [Does this decision enable or require a platform team golden path?]
* Self-service: [Can teams onboard without platform team involvement after this decision?]
* Standards and defaults: [What IaC modules, templates, or policies does this produce?]
* Observability: [Telemetry, metrics, logs, traces — unified or fragmented?]
* Incident management: [Runbooks, on-call burden, MTTR impact]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to runbooks, platform docs, or DORA metrics baseline]

---

### Pillar 6 — Sustainability

> *Resource efficiency, carbon footprint, green region selection.*

**Impact of this decision**:

* Carbon footprint: [CO₂ impact estimate; preference for low-carbon/renewable regions]
* Resource efficiency: [Scale-to-zero, right-sizing, idle elimination, shared multi-tenancy]
* Data transfer minimization: [Reducing unnecessary cross-region/cross-AZ data movement]
* Hardware lifecycle: [Shared vs. dedicated, spot reuse, e-waste considerations]
* Carbon tooling: [Measurement approach — AWS Carbon Footprint, Azure Emissions Dashboard, GCP Carbon Footprint, or cloud-agnostic tooling]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to carbon report or sustainability KPI baseline]

---

### Pillar 7 — Compliance, Governance & Data Sovereignty

> *Data residency, exit strategies, compliance frameworks, auditability, portability.*
> *This pillar is WAF++'s key extension beyond the standard six-pillar WAF.*

**Impact of this decision**:

* Data residency: [Where is data stored, processed, and replicated? Which jurisdictions apply?]
* Regulatory compliance: [GDPR, ISO 27001, SOC 2, HIPAA, PCI-DSS, BSI, NIS2, DORA, etc.]
* Sovereignty and portability: [Is the workload portable? What is the exit strategy if the vendor is changed?]
* Vendor dependency risk: [Level of lock-in; open standards vs. proprietary APIs]
* Internal governance: [Access policies, data lifecycle, approval workflows, change governance]
* Audit and evidence: [How are controls evidenced? What is the audit log trail for this decision?]
* Transparency: [Are processes and data flows documented and understandable to auditors?]

**Maturity level**: [Baseline | Standardize | Optimize] — [brief justification]

**Evidence**: [Link to compliance register, data flow diagram, or DPIA/audit report]

---

## Trade-off Summary

| Pillar | Maturity Level | Key Notes |
|---|---|---|
| Security | [Baseline / Standardize / Optimize] | |
| Cost Optimization | [Baseline / Standardize / Optimize] | |
| Performance Efficiency | [Baseline / Standardize / Optimize] | |
| Reliability | [Baseline / Standardize / Optimize] | |
| Operational Excellence | [Baseline / Standardize / Optimize] | |
| Sustainability | [Baseline / Standardize / Optimize] | |
| Compliance, Governance & Data Sovereignty | [Baseline / Standardize / Optimize] | |

**Key trade-offs**: [Describe cross-pillar tensions, e.g., "Lower vendor lock-in improves sovereignty (Pillar 7) but increases operational complexity (Pillar 5)."]

---

## Pros and Cons of the Options

### [Option A]

* Good, because [argument aligned with a WAF++ pillar]
* Good, because [argument aligned with a WAF++ pillar]
* Bad, because [which pillar is compromised and why]

### [Option B]

* Good, because [argument aligned with a WAF++ pillar]
* Bad, because [which pillar is compromised and why]

### [Option C]

* Good, because [argument aligned with a WAF++ pillar]
* Bad, because [which pillar is compromised and why]

---

## Consequences

### Positive

* [e.g., Open-source tooling eliminates vendor lock-in, improving data sovereignty]
* [e.g., GitOps automation reduces operational toil]

### Negative

* [e.g., Self-managed infrastructure requires dedicated platform engineering capacity]
* [e.g., Open-source tooling requires internal security review before adoption]

### Neutral / Follow-up decisions required

* [e.g., A separate ADR is needed for the data governance policy and retention rules]
* [e.g., Compliance review required: DPIA for cross-border data flows]
* [e.g., Exit strategy ADR: define portability requirements and migration runbook]

---

## Audit Trail

| Date | Author | Change |
|---|---|---|
| [YYYY-MM-DD] | [name/role] | Initial draft |
| [YYYY-MM-DD] | [name/role] | [e.g., Security review completed] |
| [YYYY-MM-DD] | [name/role] | [e.g., Accepted after governance board review] |

---

## Links and References

* [WAF++ Framework](https://waf2p.dev/docs/wafpp/1.0/)
* [WAF++ on GitHub](https://github.com/WAF2p/framework)
* [Link type: e.g., Supersedes] [ADR-XXXX](../XXXX-title/)
* [Link type: e.g., Related to] [ADR-YYYY](../YYYY-title/)
* [Architecture diagram or data flow diagram]
* [Compliance register or risk register entry]
