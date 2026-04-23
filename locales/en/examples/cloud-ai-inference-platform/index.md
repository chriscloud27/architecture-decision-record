# Select Cloud AI Inference Platform for Production Workloads

**Status**: Accepted

**Deciders**: Principal Platform Architect, Head of ML Engineering, Cloud Centre of Excellence Lead

**Date**: 2024-09-15

**Tags**: ai, inference, cloud, aws, azure, gcp, well-architected, cost, sustainability, performance

---

## Context and Problem Statement

Our AI-native platform (see [ADR: Adopt an AI-Native Platform Architecture](../ai-native-platform-architecture/)) requires a scalable, cost-efficient inference platform to serve production ML models. We currently run inference on a mix of ad-hoc EC2 instances and managed endpoints, resulting in inconsistent latency, high idle costs, and no unified deployment model.

We need to decide where and how to host production AI inference workloads: using cloud-provider managed inference services, self-managed Kubernetes-based serving, or a hybrid approach. This decision affects all six WAF pillars and has a significant sustainability impact given the energy intensity of GPU inference.

## Decision Drivers

* **Latency requirements**: Customer-facing inference endpoints must achieve p95 ≤ 150 ms globally.
* **Scale**: Peak inference volume of 50,000 requests/second during product launches, with typical baseline of 5,000 requests/second.
* **Cost control**: Current monthly GPU inference spend is unbudgeted and growing 30% month-over-month.
* **Multi-model serving**: 40+ production models must share infrastructure efficiently.
* **Sustainability**: GPU inference is the largest single source of cloud carbon emissions in our portfolio.
* **Portability**: Avoid deep lock-in to a single cloud provider's inference API format.

## Considered Options

* **Option A**: Self-managed Kubernetes inference serving using KServe + NVIDIA Triton Inference Server on GPU node pools (EKS/AKS/GKE)
* **Option B**: Managed inference endpoints from a single cloud provider (AWS SageMaker Inference, Azure Machine Learning Managed Endpoints, or GCP Vertex AI Prediction)
* **Option C**: Hybrid — KServe for custom/open-weight models; managed cloud endpoints for foundation models via an AI gateway

## Decision Outcome

**Chosen option**: Option C — Hybrid inference architecture using KServe + Triton for custom models, and an AI gateway routing to managed foundation model APIs.

**Rationale**: Option C provides the best total value across the WAF pillars. Custom models benefit from the GPU efficiency and multi-model packing of KServe + Triton. Foundation model inference is better served through managed APIs (Bedrock, Azure OpenAI, Vertex AI Gemini) where operational complexity and GPU management are handled by the provider. An AI gateway layer unifies routing, caching, rate limiting, and cost visibility across both paths.

---

## Well-Architected Framework Assessment

### Operational Excellence

**Impact of this decision**:

* **Monitoring and observability**: KServe emits standardized inference metrics (request rate, latency percentiles, error rate, GPU utilization) to the platform observability stack. The AI gateway adds centralized logging of all foundation model API calls, including token counts, latency, and model version. A model performance dashboard surfaces drift and accuracy signals alongside infrastructure metrics.
* **Automation and IaC**: Model deployments are declarative (KServe InferenceService manifests in Git). Canary and shadow deployments are automated via Argo Rollouts. The AI gateway is deployed via Helm with environment-specific configuration.
* **Incident response**: Automated rollback triggers on p95 latency breach or error rate spike. Foundation model API fallback routes are pre-configured in the gateway. Runbooks for GPU node failure, OOM evictions, and model loading failures are maintained and tested quarterly.
* **Team cognitive load**: Platform team owns the serving infrastructure. ML engineers interact through a simple CLI (`platform model deploy`) that abstracts Kubernetes complexity. Foundation model teams interact exclusively through the AI gateway SDK.

**Alignment**: High — declarative deployments, automated rollback, and abstraction layers minimize operational toil.

---

### Security

**Impact of this decision**:

* **Identity and access management**: KServe inference services use OIDC workload identity. Foundation model API keys are stored in cloud KMS-backed secrets (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) and injected at runtime. The AI gateway enforces API key authentication and per-consumer rate limits.
* **Data protection**: Inference request and response payloads are logged with PII redaction applied at the gateway layer before persistence. Model artifacts are stored in encrypted, private artifact registries. Inference logs are retained for 90 days for audit purposes.
* **Threat detection**: The AI gateway inspects inference payloads for prompt injection patterns (using a lightweight classifier). Anomalous usage patterns (token spike, unusual caller geography) trigger automated alerts. Container images are scanned on every build and blocked if critical CVEs are detected.
* **Compliance**: All inference traffic stays within the organization's cloud tenancy (no public internet routing for internal models). Foundation model API calls to external providers are routed via private endpoints where available (AWS PrivateLink for Bedrock, Azure Private Endpoints for Azure OpenAI).
* **AI-specific risks**: Model outputs are filtered through a content safety layer at the gateway before returning to callers. Jailbreak and harmful content detection is applied to all text generation outputs. A responsible AI review is mandatory before production deployment of any generative AI model.

**Alignment**: High — AI gateway provides a security enforcement perimeter for all inference traffic.

---

### Reliability

**Impact of this decision**:

* **Availability targets**: Inference platform SLO is 99.95% monthly. KServe deployments span multiple availability zones. Foundation model APIs include failover routing (primary → secondary provider) at the gateway layer.
* **Fault isolation**: Each model runs in its own Kubernetes namespace with dedicated resource quotas. A failing model cannot consume resources from neighbours. The AI gateway implements circuit breakers per foundation model endpoint.
* **Recovery**: Model artifacts are versioned and geo-replicated. Rollback to a previous model version completes in under 5 minutes. GPU node failure triggers automatic pod rescheduling; a spare node pool is maintained via cluster autoscaler warm buffers.
* **Resilience patterns**: Inference requests implement exponential backoff with jitter. Multi-model serving allows fallback to a smaller, faster model when the primary model is overloaded. Foundation model API timeouts are set conservatively to avoid cascade failures.
* **Dependency management**: Foundation model provider SLAs are tracked; contractual SLA is minimum 99.9%. Dependency on external providers is documented with explicit fallback strategies per model use case.

**Alignment**: High — multi-AZ deployment, circuit breakers, and warm node pools provide strong reliability.

---

### Performance Efficiency

**Impact of this decision**:

* **Scaling strategy**: KServe uses KEDA autoscaling triggered by request queue depth per model. Scale-to-zero is enabled for low-traffic models (< 10 requests/hour) to eliminate idle GPU waste. GPU node pools use cluster autoscaler with a 2-minute warm-up time, acceptable for the defined workload patterns.
* **Latency targets**: NVIDIA Triton with TensorRT-optimized models achieves p95 < 50 ms for image classification and < 100 ms for small language models on A10G GPUs. Foundation model APIs from Bedrock and Azure OpenAI achieve p95 < 1,500 ms for 1,000-token completions, meeting all defined SLOs.
* **Resource utilization**: Triton multi-model serving packs 40+ models onto shared GPU nodes, achieving 75–85% GPU utilization during business hours. Dynamic batching is enabled for all batch-compatible models, increasing throughput by 3–5×.
* **Caching and data locality**: The AI gateway implements semantic caching for foundation model responses (cache hit rate target: 25% for FAQ-type workloads). Feature serving for online models is co-located in the same availability zone as inference nodes.
* **Technology selection**: TensorRT and ONNX Runtime are used for model optimization. INT8 quantization is applied to all models where accuracy degradation is < 1%. Speculative decoding is evaluated for large language model inference to reduce latency.

**Alignment**: High — Triton multi-model serving and dynamic batching maximize GPU efficiency; semantic caching reduces foundation model costs.

---

### Cost Optimization

**Impact of this decision**:

* **Pricing model**: Custom model inference uses 1-year reserved GPU instances for baseline load (65% discount vs. on-demand) and spot instances for batch inference jobs. Foundation model API usage is pre-purchased via committed-use agreements where volume justifies it (evaluated quarterly).
* **Total cost of ownership**: Hybrid approach eliminates need to self-manage GPU infrastructure for foundation models, reducing operational engineering cost by ~2 FTEs vs. a fully self-hosted approach. Platform team investment is ~3 FTEs for the KServe/Triton platform.
* **FinOps practices**: Every inference request is tagged with `team`, `model`, `use_case`, and `environment`. The AI gateway tracks token consumption per caller and surfacing monthly spend per model and team in the FinOps dashboard. Automated alerts fire when a model's hourly cost exceeds budget thresholds.
* **Build vs. buy**: KServe and Triton are open source with no licensing cost. The AI gateway is built in-house (2-week initial build, 0.5 FTE ongoing maintenance) to avoid per-request markup from commercial AI gateway vendors.
* **Cost of inaction**: Current ad-hoc inference infrastructure costs 40% more than projected for the hybrid platform due to idle GPU instances and lack of multi-model packing.

**Alignment**: High — reserved instances, multi-model packing, semantic caching, and FinOps tagging drive significant cost reduction.

---

### Sustainability

**Impact of this decision**:

* **Carbon footprint**: GPU inference is deployed in low-carbon cloud regions: AWS eu-north-1 (Sweden, ~95% renewable), Azure northeurope (Ireland, ~80% renewable), GCP europe-north1 (Finland, ~97% renewable). Non-latency-sensitive batch inference jobs are scheduled during regional low-carbon periods using the cloud provider carbon signal APIs (AWS Carbon Intensity API, GCP Carbon-Free Energy signals).
* **Resource efficiency**: Scale-to-zero eliminates idle GPU energy use for low-traffic models. Multi-model packing on Triton increases GPU utilization from ~20% (dedicated endpoints) to 75–85%, reducing the number of active GPU nodes by 75%. Model quantization (INT8) reduces energy per inference by 30–50% on compatible models.
* **Data transfer minimization**: Inference nodes are co-located with feature stores and data sources in the same availability zone, eliminating cross-AZ data transfer charges and the associated energy use. Foundation model API calls are routed to the closest regional endpoint.
* **Hardware lifecycle**: Shared multi-tenant GPU nodes support higher utilization than dedicated per-model instances, reducing the effective physical hardware footprint. Spot instance usage reuses underutilized existing provider hardware.
* **Sustainability tooling**: Carbon metrics per inference model are reported monthly alongside cost metrics in the FinOps dashboard. Sustainability impact is a required field in every model deployment request, documenting the expected carbon cost per 1,000 inferences.

**Alignment**: High — low-carbon region selection, scale-to-zero, INT8 quantization, and GPU multi-tenancy deliver strong sustainability outcomes.

---

## Trade-off Summary

| Pillar | Alignment | Notes |
|---|---|---|
| Operational Excellence | High | Declarative deployments, automated rollback, AI gateway abstraction |
| Security | High | AI gateway security perimeter, PII redaction, content safety layer |
| Reliability | High | Multi-AZ, circuit breakers, provider failover, warm node pools |
| Performance Efficiency | High | Triton dynamic batching, TensorRT, semantic caching, KEDA autoscaling |
| Cost Optimization | High | Reserved instances, multi-model packing, FinOps tagging, AI gateway |
| Sustainability | High | Low-carbon regions, scale-to-zero, INT8 quantization, carbon reporting |

**Key trade-offs**:
* **Performance vs. Sustainability**: Real-time inference requires always-warm GPU nodes for some models, which cannot scale to zero. Mitigated by targeting ≥ 70% GPU utilization via dynamic batching and multi-model packing.
* **Portability vs. velocity**: Building an AI gateway adds initial engineering effort but provides long-term cloud portability and cost control that outweighs the 2-week delay.
* **Security vs. latency**: Content safety filtering at the gateway adds ~10 ms to generative AI response times. Accepted as a necessary trade-off for responsible AI deployment.

---

## Pros and Cons of the Options

### Option A: Self-managed KServe + Triton on GPU node pools

* Good, because maximum control over GPU optimization and model packing (Performance Efficiency)
* Good, because no per-request markup from managed service (Cost Optimization)
* Good, because full portability across cloud providers (Cost Optimization, Sustainability)
* Bad, because requires significant platform engineering expertise to operate at scale (Operational Excellence)
* Bad, because does not address foundation model serving requirements without additional complexity

### Option B: Managed inference endpoints (single cloud provider)

* Good, because reduced operational burden for infrastructure management (Operational Excellence)
* Good, because built-in security certifications and compliance (Security)
* Bad, because per-request pricing makes foundation model inference expensive at scale (Cost Optimization)
* Bad, because vendor lock-in to proprietary serving formats (Cost Optimization)
* Bad, because no cross-cloud sustainability optimization possible (Sustainability)

### Option C: Hybrid KServe + AI Gateway (chosen)

* Good, because optimizes each workload type on the best-fit infrastructure (Performance Efficiency)
* Good, because AI gateway provides unified security, cost, and sustainability visibility (Security, Cost Optimization)
* Good, because portability preserved for custom models; managed APIs used only where justified (Cost Optimization)
* Good, because semantic caching and dynamic batching reduce carbon per inference (Sustainability)
* Bad, because AI gateway is an additional component to build and maintain
* Bad, because teams must understand two deployment paths (mitigated by golden-path tooling)

---

## Consequences

### Positive

* Unified observability and cost visibility across all inference workloads via the AI gateway.
* 40% cost reduction vs. current ad-hoc infrastructure within 6 months of migration.
* Sustainable inference: estimated 60% reduction in carbon emissions per inference request from INT8 quantization and GPU multi-tenancy alone.
* Platform supports 40+ models on shared GPU infrastructure with no per-model operational overhead.

### Negative

* AI gateway requires an initial 2-week engineering investment and ongoing 0.5 FTE maintenance.
* Triton multi-model serving requires model compatibility testing; not all frameworks are supported natively.

### Neutral / Follow-up decisions required

* ADR: AI gateway technology selection (build vs. buy: LiteLLM, Portkey, custom)
* ADR: Model artifact registry and versioning strategy
* ADR: Semantic cache implementation (Redis vs. cloud-native vector store)
* ADR: Carbon scheduling policy for batch inference workloads
* Security review: AI Security Threat Model — prompt injection and content safety architecture

---

## Links and References

* [AWS Well-Architected Framework — Machine Learning Lens](https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/welcome.html)
* [Azure Well-Architected Framework — AI workloads](https://learn.microsoft.com/en-us/azure/well-architected/ai/)
* [Google Cloud Architecture Framework — AI and ML](https://cloud.google.com/architecture/framework/ai-and-ml)
* [KServe — Model Serving on Kubernetes](https://kserve.github.io/website/)
* [NVIDIA Triton Inference Server](https://developer.nvidia.com/triton-inference-server)
* [WAF ADR Template](../../templates/decision-record-template-for-well-architected-framework/)
* [Related ADR: Adopt an AI-Native Platform Architecture](../ai-native-platform-architecture/)
