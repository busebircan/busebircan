# MLOps Copilot: Roadmap

A **personal research and learning assistant** to prepare for, and grow in, a
**Senior Data Engineer / MLOps** role in insurance (Allianz).

It teaches, explains, and builds alongside you in a **personal sandbox**, so you can then do the
real work yourself at Allianz. It never connects to Allianz systems, data or code.

> Status: planning only. Nothing here is built yet.

---

## 1. Vision and scope

| Mode | What it does |
|---|---|
| **Tutor** | Explains ML fundamentals, MLOps, the tech stack and insurance ML from first principles, with cited sources |
| **Lab partner** | Builds and runs hands-on exercises with you in a local sandbox (Docker, `kind` cluster, Terraform, CI) and explains every step |
| **Reviewer** | Reviews *your* code, Dockerfiles, Helm charts, Terraform and pipelines, and explains why something is wrong |
| **Researcher** | Summarises papers, docs and regulation, and compares tools (e.g. KServe vs. Seldon, Azure ML vs. Databricks) |
| **Coach** | Study plans, quizzes, spaced repetition, system-design drills, onboarding and 30-60-90-day plans |

**Ground rules**
- Personal sandbox only: local machine, `kind`/`k3d`, and optionally a personal free-tier Azure account
- Public or synthetic data only. Never paste Allianz code, data, configs or internal documents
- **Learning first:** by default the agent explains, hints and checks your work. Full solutions only when you ask

---

## 2. Target architecture

```
                ┌────────────────────────────────────────────┐
  You  ───────▶ │  Interface: Claude Code / CLI / VS Code     │
                └──────────────────────┬─────────────────────┘
                                       ▼
                ┌────────────────────────────────────────────┐
                │  Orchestrator agent (Claude Agent SDK)      │
                │  - router: tutor | lab | review | research |│
                │    coach                                   │
                │  - teaching style: hint → explain → solve  │
                └───┬───────────┬────────────┬───────────┬───┘
                    ▼           ▼            ▼           ▼
             ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐
             │Knowledge │ │ Lessons &│ │ Sandbox   │ │ Learner      │
             │ (RAG +   │ │ labs     │ │ tools     │ │ memory       │
             │ web)     │ │ (skills) │ │ (MCP)     │ │ (progress,   │
             │          │ │          │ │           │ │ weak spots)  │
             └──────────┘ └──────────┘ └─────┬─────┘ └──────────────┘
                                             │
                      python · sql · docker · kubectl · helm · terraform · act · mlflow
                                             │
                               ┌─────────────▼─────────────┐
                               │ Local sandbox: devcontainer│
                               │ + kind cluster + Postgres │
                               │ + MLflow + (opt.) free    │
                               │ Azure subscription        │
                               └───────────────────────────┘
```

**Key design decisions**

- **LLM:** Claude via the Claude API / Agent SDK. Use a top-tier model for teaching, design
  reasoning and review, and a small fast model for quizzes and summaries.
- **Tools as MCP servers** that work only against the sandbox. You also use them in Claude Code
  while you study.
- **Lessons and labs as files:** each topic is a versioned Markdown skill with an explanation,
  an exercise, hints, a reference solution and automated checks.
- **Learner memory:** tracks which topics you've covered, quiz scores and weak spots, so the
  agent can adapt what comes next (spaced repetition).
- **Safe by construction:** no corporate credentials anywhere. Destructive commands are still
  confirmed, so you don't wipe your own lab.

**Repo layout (proposed)**

```
mlops-copilot/
├── agent/            # orchestrator, router, persona prompts, teaching style
├── mcp_servers/      # sandbox tools: python, sql, docker, k8s, terraform, ci, mlflow, docs
├── curriculum/       # lessons + labs (exercise, hints, solution, checks)
├── knowledge/        # curated sources, ingestion scripts, index config
├── projects/         # capstone projects (portfolio-ready)
├── evals/            # golden Q&A, lab-check tests
├── sandbox/          # devcontainer, kind config, docker-compose (Postgres, MLflow, Grafana)
└── .github/ + azure-pipelines.yml
```

---

## 3. Knowledge base (what the agent must know)

Curated, chunked, and indexed with source citations. Web search covers anything newer.

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
| **SQL** | Postgres, Azure SQL / Synapse, Databricks SQL, optimisation, query plans |
| **Docker** | multi-stage builds, slim/distroless images, non-root, layer caching, SBOM, Trivy scans |
| **FastAPI** | model-serving patterns, pydantic schemas, async, auth (OAuth2/Entra ID concepts), health/readiness, OpenAPI, load testing (Locust) |
| **Kubernetes** | Deployments, HPA/KEDA, probes, resource limits, ConfigMaps/Secrets, Helm, Kustomize, AKS concepts, KServe/Seldon, RBAC, NetworkPolicies, debugging (`CrashLoopBackOff`, OOMKilled) |
| **Terraform** | modules, remote state, workspaces vs. directories, `azurerm` provider, policy as code (OPA / Checkov / tfsec), drift detection |
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

> These summaries are for learning. They are not legal advice. At work, Allianz Compliance and Model Risk have the final say.

**Public datasets for practice:** `freMTPL2freq`/`freMTPL2sev` (French motor), CASdatasets (R), Kaggle insurance-fraud and churn sets, Schedule P loss triangles (CAS), synthetic data generated with SDV or Faker.

---

## 4. Sandbox tools (what the agent can do)

All tools act only on the local sandbox or your personal cloud account.

| MCP server | Tools |
|---|---|
| `python-sandbox` | run scripts/notebooks, pytest, ruff, mypy |
| `sql` | query local Postgres/DuckDB, `EXPLAIN ANALYZE`, schema introspection |
| `docker` | build, run, inspect, scan (Trivy), compare image sizes |
| `k8s` | apply/get/describe/logs on the `kind` cluster, `helm template/install`, break-and-fix scenarios |
| `terraform` | `fmt`, `validate`, `plan`, `apply` against local providers (docker, kubernetes, helm) or a personal Azure account, Checkov/tfsec |
| `ci` | lint pipeline YAML, run GitHub Actions locally with `act`, read run logs from your personal repos |
| `mlflow` | track experiments, compare runs, register models |
| `docs` | RAG search over the knowledge base + web search |
| `quiz` | generate questions, grade answers, update learner memory |

**Safety (even in a personal lab):** confirm before `destroy`/`delete`, a spending cap and
auto-teardown for any cloud resources, secrets only in a local `.env` or vault, never in prompts.

---

## 5. Curriculum: lessons + labs

Each module = explanation → guided lab → "break it and fix it" challenge → quiz → review of your solution.

| # | Module | Hands-on lab |
|---|---|---|
| 1 | Python for production | Package an ML project with uv, typing, tests, pre-commit |
| 2 | SQL & data modelling | Build a policy/claims star schema, SCD2, window-function analytics on claims |
| 3 | ML fundamentals refresher | GLM vs. GBM on `freMTPL2`, calibration, Gini, SHAP |
| 4 | Model → API | Serve the model with FastAPI (schemas, `/predict`, `/health`, versioning, tests, load test) |
| 5 | Docker | Hardened multi-stage image, non-root, Trivy scan, size optimisation |
| 6 | Kubernetes | Deploy to `kind` with Helm: probes, HPA, resources, NetworkPolicy. Debug seeded failures |
| 7 | Terraform | Modules, state, plan/apply. Provision the `kind` workloads, then (optionally) AKS + ACR + Key Vault in a personal Azure account |
| 8 | CI/CD | The same pipeline twice, in **GitHub Actions** and **Azure DevOps** YAML: lint → test → build → scan → push → deploy with environments and approvals |
| 9 | ML pipelines & registry | Validate → train → evaluate → register → promote with MLflow, orchestrated by Airflow or Prefect |
| 10 | Monitoring | Prometheus/Grafana for the API, Evidently drift reports, retraining trigger |
| 11 | Data engineering at scale | PySpark / Delta Lake ETL, data contracts, Great Expectations |
| 12 | Insurance ML deep-dives | Fraud detection (imbalance), churn + uplift, chain-ladder reserving, claims-document NLP |
| 13 | Governance | Write a model card + EU AI Act-style technical documentation for your pricing model, plus a fairness audit |
| 14 | System design | Design reviews: real-time fraud scoring, a pricing platform, a feature store, multi-team MLOps platform on AKS |

**Capstone (portfolio project):** an end-to-end motor-pricing platform built on public data:
GLM + GBM → FastAPI → Docker → Helm on Kubernetes → Terraform → GitHub Actions *and* Azure DevOps →
MLflow registry → drift monitoring → model card. It mirrors the stack you'll use at Allianz.

---

## 6. Phased delivery

### Phase 0: Foundations (week 1)
- [ ] Repo skeleton, dev container, pre-commit, CI for the copilot itself
- [ ] Sandbox: `docker-compose` (Postgres, MLflow, Grafana) + `kind` cluster script
- [ ] Interface: start inside Claude Code with MCP servers and skills
- [ ] Learner-memory format (topics, scores, notes)

### Phase 1: Tutor (weeks 2–3)
- [ ] Curate sources: official docs (K8s, TF, FastAPI, Azure, MLflow), ML texts, insurance ML papers, regulation texts
- [ ] Ingestion: chunking, embeddings, vector store (pgvector or Chroma), citations
- [ ] Personas: ML tutor, platform engineer, actuary-aware data scientist, compliance explainer
- [ ] Teaching style: Socratic hints first, explain on request, solve on request
- **Milestone:** cited, accurate explanations across all domains in §3

### Phase 2: Lab partner (weeks 4–7)
- [ ] `python-sandbox`, `sql`, `docker`, `k8s`, `terraform`, `ci` MCP servers
- [ ] Curriculum modules 1–8 with labs, hints, reference solutions and automated checks
- [ ] "Break it and fix it" scenario generator (crashlooping pods, broken pipelines, Terraform drift)
- **Milestone:** you complete modules 1–8 end-to-end with the agent checking your work

### Phase 3: ML & insurance depth (weeks 8–11)
- [ ] `mlflow` MCP server, modules 9–13
- [ ] Insurance labs on public datasets, fairness and explainability checks
- [ ] Paper/regulation research mode with structured summaries
- **Milestone:** capstone pricing model trained, explained, documented and monitored

### Phase 4: Reviewer & coach (weeks 12–14)
- [ ] Code review mode for your repos (Python, SQL, Docker, K8s, TF, pipelines)
- [ ] Quiz engine + spaced repetition from learner memory
- [ ] System-design drills (module 14) and a 30-60-90-day onboarding plan for the role
- [ ] Cert tracks if you want them: CKAD/CKA, Terraform Associate, Azure AI Engineer / DevOps Engineer
- **Milestone:** capstone finished and published as a portfolio repo

### Phase 5: Keep current (ongoing)
- [ ] Scheduled re-ingestion of docs and release notes
- [ ] A weekly digest of new MLOps tooling, insurance AI news and regulation updates
- [ ] Add modules for what you meet on the job (re-created with public or synthetic data)

---

## 7. Evaluation

| Dimension | Metric | Target |
|---|---|---|
| Explanations | Accuracy on golden Q&A (LLM-judge + manual spot checks) | ≥ 90% |
| Groundedness | Answers with valid citations | ≥ 95% |
| Labs | Reference solutions passing their own automated checks | 100% |
| Learning | Quiz scores over time, modules completed, weak topics shrinking | tracked |
| Confidence | Self-rating per topic before vs. after | tracked |

---

## 8. Risks & mitigations

| Risk | Mitigation |
|---|---|
| Hallucinated commands or config | Citations, runs everything in the sandbox, labs auto-checked |
| Learning by copy-paste | Hint-first teaching style, you write the code, the agent reviews it |
| Accidentally using Allianz material | Ground rule: public or synthetic data only. Re-create work problems generically |
| Cloud bills in the personal account | Prefer local providers, spending caps, auto-teardown |
| Stale knowledge | Scheduled re-ingestion, version-pinned docs, web search |
| Regulatory misstatements | Treated as learning summaries, cross-check against official texts |

---

## 9. Suggested build stack for the copilot

- **Agent:** Claude Agent SDK (Python), MCP servers in Python (`mcp` SDK)
- **RAG:** pgvector or Chroma, hybrid search (BM25 + embeddings) + reranking, plus web search
- **Sandbox:** dev container, `kind`/`k3d`, docker-compose (Postgres, MLflow, Prometheus, Grafana), `act` for local GitHub Actions
- **Evals:** pytest harness + LLM-as-judge + golden datasets in `evals/`
- **Interface:** Claude Code / CLI first, then a small FastAPI + web UI (which is itself a practice project)

---

## 10. Next steps

1. Review and adjust the scope, curriculum order and pace in this roadmap
2. Phase 0: repo skeleton + sandbox (`docker-compose` + `kind`)
3. Pick the first module to build the lesson format around (suggested: module 4, Model → API)
