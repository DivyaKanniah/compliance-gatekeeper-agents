# Build Roadmap

Built one agent/capability at a time. Each step ships with a runnable demo before the next one starts. Production concerns (checkpointing, reliability, observability, eval) are first-class steps, not a bolt-on at the end.

- [ ] **Step 0 — Synthetic data engine.** Procedural generator producing synthetic code/infra diffs across change types (app code, IaC, config, DB migration) with injectable violation types (secrets in code, PII in logs, missing access-control checks, vulnerable/unpinned dependency bumps, unencrypted data-at-rest) plus clean/compliant diffs. A default universal policy pack (structured YAML/JSON: control id, description, severity) and a synthetic past-incident corpus with known ground truth (doubles as the Step 9 eval set). `src/compliance_gatekeeper_agents/synthetic/`, `scripts/generate_fixtures.py`.
- [ ] **Step 1 — Change Classifier Agent.** Given a diff + metadata, decide change type and which systems/data it touches.
- [ ] **Step 2 — Diff Analysis Agent.** Summarize what actually changed from the raw diff (new deps, new endpoints, access-control changes, data-handling changes).
- [ ] **Step 3 — Policy Compliance Agent.** Given the diff summary + active policy pack, reason about which controls (if any) are violated, with confidence score and cited evidence lines.
- [ ] **Step 4 — Precedent / Incident Agent.** Vector-search past incidents for a similar-shaped issue; surface matches that support or contradict the Policy Agent's finding.
- [ ] **Step 5 — Risk Scoring & Escalation Agent.** Combine findings into a risk score + decision; auto-approve with an audit-log entry, or create an SLA-tracked human-review queue item with the full evidence trail.
- [ ] **Step 6 — Orchestrator.** Wire Steps 1–5 into a LangGraph state graph with durable checkpointing (SQLite), per-node timeout, bounded retry, a circuit breaker on the LLM client, and cost-aware model routing (small model for classification, larger for compliance reasoning).
- [ ] **Step 7 — Trigger / intake layer.** Mocked PR webhook (schema matches a real GitHub/GitLab payload) → dedup/idempotent queue → orchestrator run creation (`thread_id` = change id). Status adapter posts the result back (mocked PR comment / audit log), built against the same interface a real integration would use.
- [ ] **Step 8 — Observability.** Structured per-run trace (already available via checkpoints) plus aggregate metrics: p50/p95 latency, escalation rate, retry rate, exposed via a metrics endpoint.
- [ ] **Step 9 — Eval harness.** Regression-test the graph against the labeled synthetic diff set from Step 0 (precision/recall on violation detection), integrated with `pytest`.
- [ ] **Step 10 — Containerization & concurrency demo.** Docker Compose (app + Ollama + Chroma + queue), multiple worker replicas pulling from the queue, a concurrent-burst load test proving the design has no single-threaded bottleneck.
- [ ] **Step 11 — Policy-pack swap demo.** Add a second policy pack (e.g. BFSI: PCI-DSS, SOX, data-residency) as pure config, no code change, proving the pluggability claim.
- [ ] **Step 12 (optional) — Review-queue UI.** Minimal FastAPI + vanilla-JS view of escalated items and their evidence trail (same pattern as the `narrative-bi-agents` Boardroom UI).

Framework: LangGraph. Model host: local/self-hosted (Ollama-compatible), swappable via a thin client interface so any OpenAI-compatible endpoint (including Claude) also works.
