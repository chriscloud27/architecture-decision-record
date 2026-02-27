# Decision record template for WAF++

**WAF++** (Well-Architected Framework++) is an open-source, community-driven, vendor-neutral framework for assessing cloud and platform architectures. It extends the standard six-pillar WAF with a seventh pillar — **Compliance, Governance & Data Sovereignty** — and introduces maturity levels and built-in evidence tracing to make architecture decisions auditable.

* Website: [waf2p.dev](https://waf2p.dev/)
* Framework source: [github.com/WAF2p/framework](https://github.com/WAF2p/framework)
* License: Open source, community-governed

## The Seven Pillars of WAF++

| # | Pillar | Focus |
|---|---|---|
| 1 | Security | Controls, threat modelling, policy-as-code, secure defaults |
| 2 | Cost Optimization | FinOps, cost transparency, budgets, guardrails, right-sizing |
| 3 | Performance Efficiency | Scalability, latency, load patterns, resource tuning |
| 4 | Reliability | SLOs, backups, failover, chaos testing, DR |
| 5 | Operational Excellence | Developer experience, standards, paved road, self-service |
| 6 | Sustainability | Resource efficiency, carbon footprint, green regions |
| 7 | Compliance, Governance & Data Sovereignty | Data residency, exit strategies, compliance, portability |

## Maturity Levels

Each pillar is assessed against three maturity levels:

| Level | Description |
|---|---|
| **Baseline** | Minimum standards, clear responsibilities, operating model |
| **Standardize** | Repeatable patterns, automation, policies, documented defaults |
| **Optimize** | Measurable improvement via KPIs: cost, performance, resilience, governance |

## Why use this template

Use this template when you want:

* **Vendor-neutral** architecture decisions not anchored to a single cloud provider
* **Auditability and evidence tracing** — decisions that can be reviewed internally and externally
* **Data sovereignty** explicitly assessed — where data is processed, stored, and who controls it
* Alignment with the **WAF++ community standard** at [waf2p.dev](https://waf2p.dev/)
* Decisions that feed into a **platform governance model** rather than one-off choices

## Relationship to cloud provider WAFs

WAF++ complements rather than replaces cloud provider WAF reviews. Use both:

* **WAF++** for vendor-neutral, auditable, governance-first architecture decisions
* **AWS / Azure / GCP WAF** for cloud-provider-specific service selection and deployment reviews
