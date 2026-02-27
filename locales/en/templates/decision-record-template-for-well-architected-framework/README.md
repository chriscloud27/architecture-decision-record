# Decision record template for Well-Architected Framework

This ADR template aligns architecture decisions with the six pillars of the Well-Architected Framework (WAF), as defined by AWS, Microsoft Azure, and Google Cloud Platform:

* [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
* [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
* [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)

## The Six Pillars

| Pillar | AWS | Azure | GCP |
|---|---|---|---|
| Operational Excellence | ✓ | ✓ | ✓ |
| Security | ✓ | ✓ | ✓ |
| Reliability | ✓ | ✓ | ✓ |
| Performance Efficiency | ✓ | ✓ | ✓ |
| Cost Optimization | ✓ | ✓ | ✓ |
| Sustainability | ✓ | ✓ | ✓ |

## When to use this template

Use this template when making architecture decisions that:

* Affect cloud platform selection or service usage
* Have significant impact on operational burden, security posture, or cost
* Introduce AI/ML workloads or AI-native platform capabilities
* Involve sustainability or carbon-footprint considerations
* Require cross-pillar trade-off analysis
* Will be reviewed against a cloud provider's Well-Architected Review
