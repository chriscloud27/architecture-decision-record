# Adopt an AI-Native Platform Architecture

**Status**: Accepted

**Deciders**: Principal Platform Architect, CTO, Head of Engineering, Head of Product

**Date**: 2024-06-01

**Tags**: ai, platform, cloud, architecture, well-architected, sustainability, operational-excellence

---

## Context and Problem Statement

Our organization is undergoing a strategic transformation to embed AI capabilities across all products and operational workflows. Today our platform is built on a conventional microservices architecture with separate, bolted-on ML pipelines. This creates fragmentation: data scientists work in isolation from platform engineers, model deployments are manual and error-prone, and there is no consistent governance framework for AI models in production.

We need to decide whether to evolve the existing platform incrementally, adopt an AI-native platform pattern from the ground up, or procure a managed AI platform service from a cloud provider. The decision must be sustainable, secure, cost-efficient, and aligned with the organization's cloud strategy on AWS, Azure, and GCP.

## Decision Drivers

* **AI-first product strategy**: The organization's roadmap requires AI inference capabilities embedded in every product surface within 18 months.
* **Operational scalability**: Separate ML pipelines and ad-hoc model deployments create unacceptable toil and unreliable release cycles.
* **Governance and compliance**: Regulated industries demand auditability, explainability, and data lineage for AI outputs.
* **Cost predictability**: Uncontrolled GPU/TPU spend on experimental workloads is unsustainable.
* **Sustainability commitments**: The organization has pledged net-zero operations by 2030; AI workloads are among the highest energy consumers.
* **Talent and velocity**: Attract and retain AI engineering talent with a modern, well-documented, developer-friendly platform.

## Considered Options

* **Option A**: Adopt an AI-native internal developer platform (IDP) built on Kubernetes with MLOps tooling (Kubeflow, MLflow, Ray, Argo Workflows)
* **Option B**: Standardize on a managed AI platform from a single cloud provider (e.g., Amazon SageMaker, Azure Machine Learning, or Google Vertex AI)
* **Option C**: Incrementally extend the existing microservices platform with purpose-built AI services added over time

## Decision Outcome

**Chosen option**: Option A — AI-native internal developer platform on Kubernetes with open MLOps tooling, deployed on a primary cloud provider with a multi-cloud abstraction layer.

**Rationale**: Option A provides the best balance across all six WAF pillars. It avoids single-vendor lock-in (critical for long-term cost and sustainability), enables standardized governance, and gives developers a consistent developer experience across any cloud target. Option B is attractive for velocity but introduces deep lock-in and reduces portability. Option C defers the core architectural problem and increases long-term technical debt.

---

## Well-Architected Framework Assessment

### Operational Excellence

> *Run and monitor systems to deliver business value, and continually improve supporting processes and procedures.*

**Impact of this decision**:

* **Monitoring and observability**: The platform will expose a unified observability stack (OpenTelemetry, Prometheus, Grafana) for both application and ML-specific metrics: model drift, prediction latency, data skew, and GPU utilization. All model endpoints emit structured logs and traces by default.
* **Automation and IaC**: All platform infrastructure is defined as code using Terraform and Helm. Model training pipelines are declared via Argo Workflows YAML. Model promotion from staging to production follows a GitOps workflow with automated gate checks (accuracy, latency, bias metrics).
* **Incident response**: Runbooks are co-located with model manifests in the same repository. Automated rollback triggers on degraded model performance metrics. On-call rotation is supported by PagerDuty integration.
* **Team cognitive load**: The platform team provides a golden-path CLI and self-service templates, enabling data scientists to deploy models without deep Kubernetes expertise.

**Alignment**: High — GitOps-driven automation and embedded observability dramatically reduce operational toil.

---

### Security

> *Protect information, systems, and assets while delivering business value through risk assessments and mitigation strategies.*

**Impact of this decision**:

* **Identity and access management**: All workloads use OIDC-based workload identity (no long-lived credentials). RBAC policies enforce least privilege: data scientists can deploy to staging; production promotions require a second approver. Model training jobs cannot access production data stores directly.
* **Data protection**: Training datasets and model artifacts are encrypted at rest (AES-256) and in transit (TLS 1.3). Sensitive datasets use column-level encryption with key management via cloud KMS (AWS KMS / Azure Key Vault / GCP Cloud KMS).
* **Threat detection**: Falco runtime security monitors container behaviour. All API calls and model inference requests are logged to a central SIEM. Dependency vulnerability scanning (Trivy) runs on every container build.
* **Compliance**: The platform design supports ISO 27001, SOC 2 Type II, and GDPR requirements. Model outputs that affect individuals are subject to mandatory explainability reviews. A model card is required for every production model.
* **Network security**: All inter-service communication uses mTLS (via Istio service mesh). External AI inference endpoints are fronted by a WAF. Private endpoints are used for all cloud storage and database connections.
* **AI-specific risks**: Prompt injection, model inversion, and data poisoning are addressed in the AI Security Threat Model (separate ADR). Red-team exercises are conducted quarterly.

**Alignment**: High — security is built into the platform by design, not bolted on.

---

### Reliability

> *Ensure a workload performs its intended function correctly and consistently when it is expected to.*

**Impact of this decision**:

* **Availability targets**: Core inference APIs target 99.9% monthly availability (SLO). Training pipeline availability is best-effort with 4-hour RTO. SLOs are codified and reviewed quarterly.
* **Fault isolation**: Model serving is isolated per namespace with resource quotas. A misbehaving model cannot starve compute from other models. Each model endpoint has an independent circuit breaker.
* **Recovery**: Model artifacts are versioned in an artifact registry with geo-redundant replication. Training checkpointing allows interrupted jobs to resume. DR runbooks are tested bi-annually via game days.
* **Resilience patterns**: Inference services implement exponential backoff and jitter for upstream calls. Graceful degradation returns cached or fallback predictions when live models are unavailable.
* **Dependency management**: A service dependency map is maintained in the platform wiki and reviewed at each architecture review. All external AI API dependencies (e.g., foundation model providers) have fallback strategies.

**Alignment**: High — namespace isolation, circuit breakers, and versioned artifacts provide strong reliability guarantees.

---

### Performance Efficiency

> *Use computing resources efficiently to meet system requirements, and maintain that efficiency as demand changes and technologies evolve.*

**Impact of this decision**:

* **Scaling strategy**: Model serving scales horizontally via KEDA (Kubernetes Event-Driven Autoscaling) triggered by request queue depth. GPU nodes use cluster autoscaler with scale-to-zero for batch workloads. Spot/preemptible instances are used for training; on-demand reserved instances for inference.
* **Latency targets**: Real-time inference endpoints target p95 ≤ 200 ms. Batch inference pipelines target throughput over latency. Latency budgets are validated in load tests before every production promotion.
* **Resource utilization**: GPU utilization targets ≥ 70% during active workloads via model batching and multi-model serving on shared GPU nodes. CPU model serving uses quantized (INT8) model variants where accuracy permits.
* **Caching and data locality**: Feature store (Feast or Tecton) provides low-latency feature retrieval, co-located with inference services to minimize cross-region calls. Embedding caches reduce redundant computation for high-frequency queries.
* **Technology selection**: ONNX runtime and TensorRT are used for inference optimization. Foundation model APIs (e.g., OpenAI, Anthropic, Bedrock, Azure OpenAI) are proxied through a gateway that adds caching, rate limiting, and cost tracking.

**Alignment**: High — autoscaling, quantization, and caching ensure efficient use of expensive GPU resources.

---

### Cost Optimization

> *Avoid unnecessary costs by understanding spending over time, controlling fund allocation, and scaling to meet business needs without overspending.*

**Impact of this decision**:

* **Pricing model**: Training workloads use spot/preemptible instances (60–80% cost reduction). Inference uses a mix of reserved instances for baseline load and on-demand for burst. Scale-to-zero for batch models eliminates idle GPU spend.
* **Total cost of ownership (TCO)**: While Kubernetes adds platform complexity, the managed control plane (EKS/AKS/GKE) reduces operational overhead. The multi-cloud abstraction layer adds initial engineering cost but provides negotiating leverage and avoids lock-in premiums.
* **FinOps practices**: All resources are tagged by team, model, environment, and cost centre. Monthly FinOps reviews compare actual vs. budgeted spend per model. Automated rightsizing recommendations via cloud provider tools (AWS Compute Optimizer, Azure Advisor, GCP Recommender) are reviewed quarterly.
* **Build vs. buy**: Core MLOps tooling (Kubeflow, MLflow, Ray) is open source. Managed AI platform services (SageMaker, Vertex AI) are used selectively for specialised capabilities (e.g., AutoML, foundation model fine-tuning) rather than as the primary platform.
* **Cost of inaction**: Continuing with ad-hoc ML deployments will result in duplicated infrastructure, untracked GPU spend, and a growing gap between AI investment and business value realization.

**Alignment**: High — FinOps practices, spot instances, and scale-to-zero significantly reduce AI infrastructure costs.

---

### Sustainability

> *Minimize the environmental impact of running cloud workloads by reducing energy consumption and improving efficiency across all components.*

**Impact of this decision**:

* **Carbon footprint**: Primary cloud regions are selected based on renewable energy availability (AWS eu-north-1 Stockholm, Azure northeurope, GCP europe-north1 Finland — all > 90% renewable). Workload placement policy prefers low-carbon regions for non-latency-sensitive training jobs.
* **Resource efficiency**: Scale-to-zero eliminates idle GPU/CPU waste outside business hours. Model compression (quantization, distillation) reduces compute per inference by 30–70%. Batch inference is preferred over real-time where business requirements allow, enabling higher GPU utilization and fewer idle resources.
* **Data transfer minimization**: Feature stores and artifact registries are co-located with compute to minimize cross-region data movement. Inference endpoints are deployed close to users (edge/regional) to avoid unnecessary round-trips.
* **Hardware lifecycle**: Kubernetes shared infrastructure allows higher multi-tenancy than dedicated VM fleets, reducing physical hardware footprint. Spot instances reuse existing cloud provider hardware, avoiding incremental provisioning.
* **Sustainability tooling**: Monthly carbon reports are generated using the AWS Customer Carbon Footprint Tool, Azure Emissions Impact Dashboard, and GCP Carbon Footprint. Carbon metrics are surfaced in the engineering FinOps dashboard alongside cost metrics. A sustainability champion is assigned per team to review and act on recommendations.

**Alignment**: High — renewable region selection, scale-to-zero, model compression, and carbon tooling provide strong sustainability alignment.

---

## Trade-off Summary

| Pillar | Alignment | Notes |
|---|---|---|
| Operational Excellence | High | GitOps, golden-path templates, and automated rollback reduce toil |
| Security | High | Zero-trust, workload identity, mTLS, and AI-specific threat modelling |
| Reliability | High | Namespace isolation, circuit breakers, geo-redundant artifacts |
| Performance Efficiency | High | GPU autoscaling, quantization, feature store, inference caching |
| Cost Optimization | High | Spot instances, scale-to-zero, FinOps tagging and reviews |
| Sustainability | High | Renewable regions, model compression, carbon tooling |

**Key trade-offs**:
* **Complexity vs. control**: Option A requires a more sophisticated platform engineering capability than Option B. This is mitigated by golden-path tooling and a dedicated platform team.
* **Velocity vs. portability**: Option B (managed cloud AI platform) would accelerate initial delivery but creates deep lock-in. Option A takes 2–3 months longer to bootstrap but pays back within the first year.
* **Reliability vs. sustainability**: Multi-region active-active improves reliability but increases carbon footprint. An active-passive regional design is used for non-critical workloads to balance these pillars.

---

## Pros and Cons of the Options

### Option A: AI-native IDP on Kubernetes (chosen)

* Good, because open-source MLOps stack avoids cloud vendor lock-in (Cost Optimization, Sustainability)
* Good, because unified observability and GitOps automation reduce operational toil (Operational Excellence)
* Good, because namespace isolation and circuit breakers provide strong fault isolation (Reliability)
* Good, because scale-to-zero and spot instances optimize GPU cost (Cost Optimization)
* Good, because renewable region selection and model compression support sustainability targets (Sustainability)
* Bad, because Kubernetes and MLOps tooling requires a skilled platform engineering team (Operational Excellence — mitigated by golden-path templates)
* Bad, because initial platform bootstrap takes 2–3 months longer than Option B

### Option B: Managed cloud AI platform (single vendor)

* Good, because managed services reduce platform engineering overhead (Operational Excellence)
* Good, because built-in compliance certifications accelerate security reviews (Security)
* Bad, because deep lock-in to a single cloud provider reduces negotiating leverage (Cost Optimization)
* Bad, because lack of portability conflicts with multi-cloud sustainability region strategy (Sustainability)
* Bad, because proprietary APIs create significant migration cost if the vendor changes pricing or capabilities

### Option C: Incremental extension of existing platform

* Good, because lowest initial disruption to existing teams
* Bad, because it perpetuates fragmentation between ML and platform engineering (Operational Excellence)
* Bad, because technical debt compounds over time, increasing long-term cost (Cost Optimization)
* Bad, because no unified governance framework for AI models in production (Security)

---

## Consequences

### Positive

* A single, consistent developer experience for all AI/ML workloads across the organization.
* Automated governance gates (model cards, bias checks, accuracy thresholds) embedded in every model promotion workflow.
* Carbon and cost metrics are visible alongside each other, enabling informed sustainability decisions.
* Platform is cloud-agnostic at the workload level, enabling future multi-cloud or cloud migration without re-engineering applications.

### Negative

* Requires hiring or upskilling 3–4 platform engineers with Kubernetes and MLOps expertise within Q1.
* Initial platform build requires a 10-week investment before the first production model can be deployed via the new path.

### Neutral / Follow-up decisions required

* ADR: Foundation model selection and AI gateway strategy (prompt routing, rate limiting, cost tracking)
* ADR: Feature store technology selection (Feast vs. Tecton vs. cloud-native)
* ADR: AI governance and model card standard
* ADR: Observability stack (OpenTelemetry collector, metrics backend, tracing backend)
* Security review: AI Security Threat Model (prompt injection, model inversion, supply chain)

---

## Links and References

* [AWS Well-Architected Framework — Machine Learning Lens](https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/welcome.html)
* [Azure Well-Architected Framework — AI workloads](https://learn.microsoft.com/en-us/azure/well-architected/ai/)
* [Google Cloud Architecture Framework — AI and ML](https://cloud.google.com/architecture/framework/ai-and-ml)
* [WAF ADR Template](../../templates/decision-record-template-for-well-architected-framework/)
* [CNCF MLOps Landscape](https://landscape.cncf.io/card-mode?category=machine-learning&grouping=category)
