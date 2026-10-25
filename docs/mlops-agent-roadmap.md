# MLOps Copilot: Roadmap

A personal AI assistant for a **Senior Data Engineer / MLOps** role in insurance (Allianz).
It should be able to **explain** (ML, insurance, the stack), **generate** (code, infra, pipelines),
**operate** (run, inspect, debug, with guardrails) and **review** (code, models, compliance).

> Status: planning only. Nothing here is built yet.

---

## 1. Vision and scope

| Capability | What "done" looks like |
|---|---|
| **Expert Q&A** | Grounded, cited answers on ML fundamentals, MLOps, the tech stack and insurance ML |
| **Builder** | Scaffolds production-grade FastAPI services, Dockerfiles, Helm charts, Terraform modules, and Azure DevOps / GitHub Actions pipelines that pass lint and tests |
| **Operator** | Runs Python and SQL, builds images, inspects Kubernetes, runs `terraform plan`, reads CI logs. Write actions need explicit approval |
| **Reviewer** | Reviews PRs, IaC, and model cards for bugs, security, cost, and regulatory gaps |
| **Insurance domain expert** | Pricing, claims, fraud, reserving, churn, and the regulation that applies (EU AI Act, GDPR, Solvency II, BaFin/EIOPA) |
| **Mentor** | Onboarding plans, study tracks, interview and design-review prep |

**Out of scope for v1:** autonomous production changes, handling real customer PII, replacing actuarial sign-off.

---

## 2. Target architecture

```
                ┌──────────────────────────────────────────────┐
  You  ───────▶ │  Interface: CLI / VS Code / Slack/Teams / Web │
                └──────────────────────┬───────────────────────┘
                                       ▼
                ┌──────────────────────────────────────────────┐
                │  Orchestrator agent (Claude Agent SDK)        │
                │  - router: explain | build | operate | review │
                │  - planner + tool-use loop                    │
                │  - guardrails: approval gates, policy checks  │
                └───┬───────────┬────────────┬────────────┬────┘
                    ▼           ▼            ▼            ▼
             ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐
             │Knowledge │ │ Skills   │ │ Tools/MCP │ │ Memory    │
             │ (RAG)    │ │(playbooks│ │ servers   │ │ & audit   │
             │          │ │templates)│ │           │ │ log       │
             └──────────┘ └──────────┘ └───────────┘ └───────────┘
                                             │
              python · sql · docker · kubectl · helm · terraform · az devops · gh · mlflow
                                             │
                               ┌─────────────▼─────────────┐
                               │ Sandbox (dev container /  │
                               │ kind cluster / dev sub)   │
                               └───────────────────────────┘
```

**Key design decisions**

- **LLM:** Claude via the Claude API / Agent SDK. Use the top-tier model for planning and review,
  and a small fast model for routing and summarisation. **Check Allianz's AI usage policy first.**
  If external APIs aren't allowed, use an approved enterprise channel (e.g. Claude on Microsoft
  Foundry / Azure, AWS Bedrock or Vertex AI) so data stays inside the approved boundary.
- **Tools as MCP servers.** One server per domain (k8s, terraform, ci, data, ml-registry) so they
  can be reused across Claude Code, the IDE and chat.
- **Skills as files.** Each playbook (e.g. "deploy a model to AKS") is a versioned Markdown skill
  with templates. Easy to review and extend.
- **Read by default, write only with approval.** Every mutating command shows a dry-run or diff
  first.
- **Everything runs in a sandbox** (dev container + local `kind`/`k3d` cluster + a dev Azure
  subscription). No production credentials in v1.

**Repo layout (proposed)**

```
mlops-copilot/
├── agent/            # orchestrator, router, prompts, guardrails
├── mcp_servers/      # k8s, terraform, cicd, data, mlflow, docs
├── skills/           # playbooks + templates (fastapi, docker, helm, tf, pipelines)
├── knowledge/        # curated sources, ingestion scripts, index config
├── evals/            # golden Q&A, build tasks, safety tests
├── sandbox/          # devcontainer, kind config, sample data, demo models
├── infra/            # terraform for the copilot itself (optional)
└── .github/ / azure-pipelines.yml
```

---

## 3. Knowledge base (what the agent must know)

Curated, chunked, and indexed with source citations. Every answer cites its sources.

### 3.1 ML fundamentals
- Statistics & probability: distributions (Poisson, Gamma, Tweedie, Negative Binomial), MLE, Bayesian basics, hypothesis testing, bootstrap
- Supervised learning: linear/logistic regression, **GLMs**, trees, RF, **GBMs (XGBoost/LightGBM/CatBoost)**, SVM, kNN
- Unsupervised: clustering, PCA, anomaly detection (Isolation Forest, autoencoders)
- Deep learning: MLPs, CNNs, RNNs/Transformers, embeddings, fine-tuning, LLMs and RAG
- Model evaluation: CV strategies (time-based!), ROC-AUC, PR-AUC, log-loss, calibration, Gini / lift / Lorenz curves, deviance, RMSE/MAE
- Feature engineering, leakage, class imbalance, hyperparameter tuning (Optuna)
- Explainability: SHAP, PDP/ICE, monotonic constraints, surrogate models
- Fairness: demographic parity, equalised odds, proxy discrimination
- Time series & survival analysis (lapse, time-to-claim)
- Causal inference basics (uplift for retention campaigns)

### 3.2 MLOps & data engineering
- ML lifecycle, MLOps maturity levels (Google / Microsoft models)
- Experiment tracking & registry: **MLflow**, Azure ML
- Feature stores (Feast, Databricks / Azure ML feature store), point-in-time correctness
- Orchestration: Airflow, Azure Data Factory, Databricks Workflows, Kubeflow / Argo
- Data quality & contracts: Great Expectations, Pandera, dbt tests
- Serving patterns: batch, online (FastAPI), streaming, shadow / canary / A-B, champion–challenger
- Monitoring: data drift, concept drift, PSI / KS, performance decay, Evidently, Prometheus/Grafana
- Data modelling & SQL: window functions, CTEs, query plans, partitioning, SCD Type 2, lakehouse (Delta / Iceberg), Spark/PySpark

### 3.3 Tech stack deep-dives
| Area | Topics |
|---|---|
| **Python** | packaging (uv/poetry), typing, pytest, ruff/mypy, async, pydantic v2, performance |
| **SQL** | Postgres, Azure SQL / Synapse, Databricks SQL, Snowflake-style patterns, optimisation |
| **Docker** | multi-stage builds, slim/distroless images, non-root, layer caching, SBOM, Trivy scans |
| **FastAPI** | model-serving patterns, pydantic schemas, async, auth (OAuth2/Entra ID), health/readiness, OpenAPI, load testing (Locust) |
| **Kubernetes** | Deployments, HPA/KEDA, probes, resource limits, ConfigMaps/Secrets, Helm, Kustomize, AKS, KServe/Seldon, RBAC, NetworkPolicies, debugging (`CrashLoopBackOff`, OOMKilled) |
| **Terraform** | modules, remote state (Azure Storage), workspaces vs. directories, `azurerm` provider, policy as code (OPA / Checkov / tfsec), drift detection |
| **CI/CD** | Azure DevOps YAML (stages, templates, environments, approvals, service connections), GitHub Actions (reusable workflows, OIDC to Azure, environments), GitOps (Argo CD / Flux) |
| **Azure** | AKS, ACR, Key Vault, Azure ML, Storage/ADLS, Entra ID, Managed Identity, Monitor/Log Analytics |

### 3.4 Insurance domain ML
| Use case | Typical methods | Key concerns |
|---|---|---|
| **Pricing (P&C, motor, home)** | Frequency–severity GLMs (Poisson × Gamma), Tweedie GLM/GBM, GAMs, GBM + GLM hybrids | Interpretability, monotonicity, rate filing, price-walking bans, fairness |
| **Underwriting / risk selection** | GBMs, scorecards, external data enrichment | Explainability, high-risk under EU AI Act for life/health |
| **Claims triage & fraud** | Anomaly detection, supervised GBMs, graph/network analytics, NLP on claim notes | Class imbalance, label delay, investigator feedback loop |
| **Reserving** | Chain ladder, Bornhuetter–Ferguson, Mack; ML/individual-claim reserving | Actuarial sign-off, Solvency II, IFRS 17 |
| **Churn / retention / CLV** | Survival models, uplift models, GBMs | Causal validity, campaign measurement |
| **Document & NLP** | OCR + LLM extraction (claims, policies, medical reports), classification, RAG for underwriters | PII, hallucination controls, human in the loop |
| **Telematics / UBI** | Time-series features, driving scores | Data volume, streaming, consent |
| **Nat-cat & climate** | Geospatial features, cat-model integration | Exposure data quality |
| **Customer service** | Intent classification, chatbots, next-best-action | Conduct risk, transparency |

**Regulation & governance pack**
- **EU AI Act:** AI used for risk assessment and pricing in **life and health insurance** is *high-risk* (Annex III). Covers risk management, data governance, technical documentation, logging, human oversight, accuracy/robustness.
- **GDPR:** lawful basis, Art. 22 automated decisions, DPIAs, data minimisation, right to explanation
- **Solvency II**, **IFRS 17**, **EIOPA** guidance on AI governance, **BaFin** supervisory principles for algorithmic decision-making, **DORA** (ICT/operational resilience)
- **Gender-neutral pricing** (EU Test-Achats ruling), proxy discrimination
- Model risk management: model inventory, validation, model cards, change management

> These summaries guide design. They are not legal advice. Always confirm with Allianz Compliance and Model Risk.

---

## 4. Tooling (what the agent can do)

| MCP server | Read-only tools (v1) | Gated write tools (v2+) |
|---|---|---|
| `python-sandbox` | run scripts/notebooks, pytest, ruff, mypy | none |
| `sql` | query dev DBs, `EXPLAIN`, schema introspection | DDL in sandbox only |
| `docker` | build, scan (Trivy), inspect, run locally | push to dev ACR |
| `k8s` | get/describe/logs/events, `helm template`, `kubectl diff` | apply / rollout to sandbox namespace |
| `terraform` | `fmt`, `validate`, `plan`, Checkov/tfsec | `apply` to dev subscription |
| `cicd` | read Azure DevOps / GH Actions runs and logs, lint YAML | trigger pipelines, open PRs |
| `mlflow` | list experiments/runs/models, compare metrics | register / transition model stage |
| `docs` | RAG search over the knowledge base | none |
| `monitoring` | query Prometheus / Log Analytics, drift reports | none |

**Guardrails**
- Allow-list of commands; deny `delete`, `destroy` and prod contexts by default
- Approval prompt with dry-run/diff before every mutation
- Secrets only via Key Vault / env injection. Never in prompts or logs
- PII redaction filter on all tool output before it reaches the model
- Full audit log (prompt, tool call, result, approver)

---

## 5. Skills (reusable playbooks + templates)

1. **Model → API:** wrap a trained model in FastAPI (pydantic schemas, `/predict`, `/health`, versioning, tests)
2. **Containerise:** hardened multi-stage Dockerfile + Trivy scan + SBOM
3. **Deploy to K8s:** Helm chart with probes, HPA, resources, NetworkPolicy; canary via Argo Rollouts / KServe
4. **Infra:** Terraform modules for AKS, ACR, Key Vault, Storage, Azure ML workspace, with remote state
5. **CI/CD:** paired templates for **Azure DevOps** and **GitHub Actions**: lint → test → build → scan → push → deploy (dev → staging → prod with approvals), OIDC/workload identity
6. **Training pipeline:** data validation → feature build → train → evaluate → register → promote (MLflow)
7. **Monitoring:** drift + performance dashboards, alert rules, retraining triggers
8. **Data pipeline:** SQL/PySpark ETL with tests, data contracts, SCD2
9. **Incident runbooks:** pod crashloop, latency spike, drift alert, failed pipeline, Terraform drift
10. **Model documentation:** model card + EU AI Act technical-documentation template + validation report
11. **Insurance starter kits:** frequency–severity pricing (GLM vs. GBM), fraud detection, churn/uplift, chain-ladder reserving on public datasets (e.g. `freMTPL2`, CASdatasets)
12. **Code / PR review:** checklist for Python, SQL, Docker, K8s, TF and pipelines (security, cost, reliability)

---

## 6. Phased delivery

### Phase 0: Foundations (week 1)
- [ ] Confirm Allianz AI/LLM usage policy and approved model access path
- [ ] Repo skeleton, dev container, pre-commit, CI for the copilot itself
- [ ] Choose interface (start with CLI / Claude Code + MCP. Add Teams or web later)
- [ ] Define success metrics and the eval harness skeleton

### Phase 1: Expert Q&A, the knowledge core (weeks 2–3)
- [ ] Curate sources: official docs (K8s, TF, FastAPI, Azure, MLflow), ML textbooks/notes, insurance ML papers, regulation texts
- [ ] Ingestion pipeline: chunking, embeddings, vector store (pgvector / Azure AI Search), metadata and citations
- [ ] Router + system prompts per persona (ML tutor, platform engineer, actuary-aware data scientist, compliance reviewer)
- [ ] Eval set: ~150 golden questions across the domains. Track groundedness and accuracy
- **Milestone:** cited, accurate answers on ML fundamentals, stack and insurance topics

### Phase 2: Builder (weeks 4–6)
- [ ] Skills 1–5 with templates and generated tests
- [ ] `python-sandbox`, `docker`, `terraform` (plan only), `k8s` (read + `helm template`) MCP servers
- [ ] Self-verification loop: generate, then lint/test/plan, then fix until green
- **Milestone:** "Deploy this sklearn model to AKS via Azure DevOps" produces a working repo that passes CI in the sandbox

### Phase 3: Operator (weeks 7–9)
- [ ] Local `kind` cluster + dev Azure subscription sandbox
- [ ] `cicd`, `mlflow`, `monitoring`, `sql` MCP servers
- [ ] Approval-gated writes, audit log, PII redaction
- [ ] Runbook skill (9). Log/event triage, root-cause summaries
- **Milestone:** the agent diagnoses a broken deployment or failed pipeline and proposes a fix you can apply with one approval

### Phase 4: Insurance domain pack (weeks 10–12)
- [ ] Starter kits (skill 11) on public insurance datasets
- [ ] Fairness and explainability tooling (SHAP, monotonic GBMs, proxy-discrimination checks)
- [ ] Governance skill (10): model card + EU AI Act documentation generator
- **Milestone:** end-to-end motor-pricing demo: GLM vs. GBM, explainability, model card, deployed and monitored

### Phase 5: Reviewer & governance (weeks 13–14)
- [ ] PR review mode for GitHub and Azure DevOps (skill 12)
- [ ] Policy-as-code checks (Checkov, OPA, Trivy) explained in plain language
- [ ] Compliance review mode: flag high-risk use cases, missing documentation, data issues
- **Milestone:** automated review comments on IaC/ML PRs with few false positives

### Phase 6: Mentor & productivity (ongoing)
- [ ] 30-60-90-day onboarding plan generator for the Allianz role
- [ ] Spaced-repetition study tracks and quizzes (CKA/CKAD, Terraform Associate, Azure AI/DevOps certs)
- [ ] System-design interview / design-review simulator (e.g. "design a real-time fraud scoring platform")
- [ ] Personal memory: your team's conventions, glossary, past decisions

### Phase 7: Hardening (when moving toward real use)
- [ ] SSO (Entra ID), RBAC on tools, per-environment credentials via Managed Identity
- [ ] Observability for the agent: traces, token cost, latency, tool error rates
- [ ] Red-teaming: prompt injection via logs/docs, secret exfiltration, destructive commands
- [ ] Deploy the copilot itself with its own stack (FastAPI + Docker + AKS + Terraform + pipelines), which doubles as a portfolio project

---

## 7. Evaluation & success metrics

| Dimension | Metric | Target |
|---|---|---|
| Knowledge | Accuracy on golden Q&A (LLM-judge + manual spot checks) | ≥ 90% |
| Groundedness | Answers with valid citations | ≥ 95% |
| Build | Generated projects passing lint + tests + `terraform validate` on first try / after self-fix | ≥ 70% / ≥ 95% |
| Operate | Correct root cause on seeded incident scenarios | ≥ 80% |
| Safety | Unapproved mutations, secrets/PII leaked | 0 |
| Usefulness | Time saved per task, weekly usage | tracked |

Evals run in CI on every prompt, skill or tool change.

---

## 8. Risks & mitigations

| Risk | Mitigation |
|---|---|
| Corporate data / LLM policy blocks usage | Clear it in Phase 0. Use an approved enterprise model endpoint, synthetic or public data only |
| Hallucinated commands or config | RAG with citations, self-verification (lint/plan/test), dry-runs |
| Destructive actions | Read-only default, allow-lists, approval gates, no prod credentials |
| Prompt injection via logs, docs or PRs | Treat tool output as data, sanitise it, restrict tool scope |
| Stale knowledge | Scheduled re-ingestion, version-pinned docs |
| Regulatory misstatements | Compliance pack reviewed by a human, explicit "not legal advice" |

---

## 9. Suggested build stack for the copilot

- **Agent:** Claude Agent SDK (Python), MCP servers in Python (`mcp` SDK)
- **RAG:** pgvector or Azure AI Search. Hybrid search (BM25 + embeddings) + reranking
- **Sandbox:** dev container, `kind`/`k3d`, LocalStack-style mocks where needed, dev Azure subscription
- **Evals:** pytest-based harness + LLM-as-judge + golden datasets in `evals/`
- **Interface:** Claude Code / CLI first, then VS Code, then a Teams bot or FastAPI web UI
- **CI:** GitHub Actions *and* an Azure DevOps mirror pipeline, so the copilot repo also demonstrates both

---

## 10. Next steps

1. Review and adjust scope and priorities in this roadmap
2. Check Allianz's AI usage policy and the approved model access path
3. Start Phase 0: repo skeleton + dev container + eval harness
