# MLOps Lab

A personal study lab with Claude Code as coach, for insurance MLOps.

## Setup
1. Install: [Claude Code](https://code.claude.com), Docker, [kind](https://kind.sigs.k8s.io), `kubectl`, `helm`, Terraform, [uv](https://docs.astral.sh/uv/)
2. Copy this folder to your machine, then run `git init` (optional, but gives you history)
3. Start the sandbox:
   ```bash
   docker compose -f sandbox/docker-compose.yml up -d
   kind create cluster --config sandbox/kind-config.yaml
   ```
   MLflow: http://localhost:5000 · Grafana: http://localhost:3000 (admin / lab) · Prometheus: http://localhost:9090
4. Run `claude` in this folder and say "let's start"

## What's inside
| Path | Purpose |
|---|---|
| `CLAUDE.md` | Coach instructions: role, ground rules, teaching style, curriculum, insurance context |
| `progress.md` | Your progress, weak topics and quiz log (Claude updates it) |
| `.claude/skills/model-to-api/` | Module 4 lesson and lab |
| `.claude/skills/quiz/` | Quizzes with spaced repetition |
| `sandbox/` | Local Postgres, MLflow, Prometheus, Grafana and a `kind` cluster config |
| `references/` | Public documents for Claude to read |
| `labs/` | Your lab code, one folder per module |

## Useful prompts
- "Let's start" / "What's next?"
- "Start module 4"
- "I'm stuck, give me a hint"
- "Check my work"
- "Quiz me on Kubernetes"
- "Wrap up"

## Stop the sandbox
```bash
docker compose -f sandbox/docker-compose.yml down   # add -v to delete data
kind delete cluster --name mlops-lab
```
