---
name: model-to-api
description: Module 4 lab. Train a motor-insurance claim-frequency model on freMTPL2 and serve it with FastAPI (schemas, /predict, /health, versioning, tests). Use when I start or continue module 4, or ask how to serve a model as an API.
---

# Module 4: Model → API

Teach this module in the style set in `CLAUDE.md`: hint first, I write the code, you check it.
Work in `labs/04-model-to-api/`.

## Learning objectives
By the end I can:
- Explain batch vs. online serving and when an insurer uses each (renewal repricing vs. quote-time pricing)
- Package a trained model with its preprocessing so training and serving can't drift apart
- Build a FastAPI service with validated input/output schemas, health and readiness endpoints, and a model version
- Handle **exposure** correctly when serving a frequency model
- Test the service and load-test it

## Concepts to explain (briefly, before the lab)
1. **Serving patterns:** batch, online, streaming. Latency vs. throughput.
2. **Train/serve skew:** why preprocessing goes in a single `Pipeline` saved with the model.
3. **Schemas as contracts:** Pydantic models, enums for categoricals, value ranges (e.g. `DrivAge` 18–100).
4. **Frequency models and exposure:** the model predicts a claim *rate* per policy-year. Expected claims = rate × exposure. Training uses `ClaimNb / Exposure` as the target with `Exposure` as sample weight (or a Poisson GLM with a `log(Exposure)` offset).
5. **Health vs. readiness:** liveness ("process is up") vs. readiness ("model is loaded, can serve").
6. **Model versioning:** expose the model version in responses and logs for traceability (needed for audit and the EU AI Act logging requirements).

## Lab steps
1. **Setup:** `uv init`, add `scikit-learn pandas fastapi uvicorn pydantic joblib pytest httpx`. Optional: `mlflow`, `lightgbm`.
2. **Data:** load freMTPL2freq with `sklearn.datasets.fetch_openml(data_id=41214, as_frame=True)`. Inspect columns: `ClaimNb, Exposure, Area, VehPower, VehAge, DrivAge, BonusMalus, VehBrand, VehGas, Density, Region`.
3. **Train (`train.py`):**
   - Target `ClaimNb / Exposure`, sample weight `Exposure`
   - `ColumnTransformer` (one-hot categoricals, log `Density`, bin or spline ages) + `PoissonRegressor`
   - Report Poisson deviance on a held-out set vs. a constant-mean baseline
   - Save the pipeline with `joblib`, plus a `model_meta.json` (version, training date, metrics, feature list)
   - Optional: log the run and model to MLflow at `http://localhost:5000`
4. **Serve (`app/main.py`):**
   - Load the model once at startup (FastAPI `lifespan`)
   - `POST /predict`: request schema with all features + `Exposure` (0 < x ≤ 1). Response: `claim_frequency`, `expected_claims`, `model_version`
   - Support a batch of policies in one request
   - `GET /health` (liveness) and `GET /ready` (503 until the model is loaded)
5. **Test (`tests/`):** `TestClient` tests for a valid prediction, invalid input (422), batch requests, and `/ready` before and after loading.
6. **Run:** `uvicorn app.main:app --reload`, open `/docs`, try requests with `curl`.
7. **Stretch:** add a Prometheus `/metrics` endpoint (`prometheus-fastapi-instrumentator`) and load-test with Locust.

## Definition of done (check these by running them)
- [ ] `pytest` passes
- [ ] Held-out deviance is better than the constant-mean baseline
- [ ] Invalid input (e.g. `DrivAge: 12`, unknown `VehGas`) returns 422 with a clear message
- [ ] `expected_claims` doubles when `Exposure` goes from 0.5 to 1.0 with all else equal
- [ ] `/ready` returns 503 when the model file is missing
- [ ] Every response includes `model_version`

## Break it and fix it
After the lab passes, introduce one of these without telling me which, and let me debug it:
1. Preprocessing done in the API instead of inside the saved pipeline, with one transform subtly different
2. Exposure ignored at serving time, so `expected_claims` equals the rate
3. A categorical level unseen in training (`OneHotEncoder` without `handle_unknown="ignore"`)
4. The model loaded on every request (latency spike under load)

## Quiz (ask 3–5, then log the results in progress.md)
- Why use exposure as a weight (or offset) instead of predicting `ClaimNb` directly?
- What is the difference between `/health` and `/ready`, and what does Kubernetes do with each?
- How would you serve 2 million renewal prices overnight? Would you use this API?
- How do you guarantee the same preprocessing at training and serving time?
- Why does an insurer need `model_version` in every prediction log?

## Reference solution
Only when I say "show me the solution": write it into `labs/04-model-to-api/solution/` and walk me through the differences from my version.
