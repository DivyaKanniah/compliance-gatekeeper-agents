# CLAUDE.md

Instructions for Claude Code when working in this repository.

## This is a learning project — explain as you build

This repo exists for hands-on learning, not just to ship a demo. **Every step, always explain in short what you're doing and why before/while doing it** — what the flow is, what agent or piece is being added, how it fits into the graph, and any production concern it demonstrates (checkpointing, retry, memory tier, etc.). Keep explanations short (a few sentences), not essays — the goal is the user builds an accurate mental model of each mechanism as it's added, not that they read documentation afterward. Don't skip this even for "obvious" steps — the obvious steps are often exactly what needs reinforcing.

## What this project is

A multi-agent system (`compliance-gatekeeper-agents`) that reviews code/infrastructure changes against a **pluggable** security & compliance policy pack, auto-approving low-risk changes and escalating the rest to a human reviewer with a full evidence trail. Full architecture and agent responsibilities: [`README.md`](README.md). Build order and current status: [`docs/ROADMAP.md`](docs/ROADMAP.md) — always check this before starting work, and keep it updated as steps complete.

This project's purpose is dual:
1. **Learning vehicle** — hands-on reps on the specific mechanics of production agentic systems: LangGraph state/checkpointing, dynamic conditional routing (not a fixed pipeline), tool use with real side effects, retry/circuit-breaker reliability, observability, evals.
2. **Portfolio piece** — intended for GitHub, so no real employer/client data or code anywhere, ever. All diffs, policy packs, and incident data are synthetic (Step 0).

## Core design principles (do not violate these while building)

- **Policy packs are config, not code.** The Policy Compliance Agent reads an active policy pack (YAML/JSON); adding or swapping a pack (e.g. a BFSI-specific one) must never require touching agent logic. This is the pluggability claim the project is built to prove — don't quietly hardcode a policy check into Python.
- **Mocked integrations are built to the real interface.** The PR-webhook trigger and the result/status adapter are mocked (no real GitHub/Jira access), but their payload shapes mirror what the real integration would send/expect, so swapping in the real one later is an adapter swap, not a rewrite. Don't take shortcuts that would break this.
- **Memory has two tiers, and they are not interchangeable.** Short-term = LangGraph state for one change's run (diff, findings, retry count, trace), checkpointed, dies with the run. Long-term = external stores (vector store of past incidents, the policy pack itself) that agent nodes call into like any external API — never fold long-term data into the graph state schema.
- **Never auto-resolve an ambiguous security finding.** Below the confidence threshold, escalate to the human-review queue. Do not add logic that guesses past that boundary, even to make a demo look more automated.
- **No code execution.** This system analyzes and reasons about diffs; it does not execute LLM-generated or reviewed code. If a future step is tempted to add execution (e.g. to verify a fix), stop and flag it — that reintroduces a sandboxing problem this project has deliberately avoided.
- **Design for scale, don't fake running at scale.** Queue-based intake, durable checkpointing, bounded retries, and stateless workers should all be real. Demonstrating concurrency (Step 10) means proving the design has no single-threaded bottleneck at small N — not claiming production-scale load that was never actually run.

## Stack

Python, LangGraph (orchestration), Ollama (local LLM, swappable client interface), FastAPI, Chroma (vector store), a queue for intake (Redis or SQLite-backed), Docker Compose for the full local stack, pytest for the eval harness.

## Conventions

- Package lives under `src/compliance_gatekeeper_agents/`, one module per agent under `agents/`, graph wiring under `graph/`, synthetic data generation under `synthetic/`.
- No real data of any kind — everything under `data/synthetic/` is procedurally generated and safe to commit.
- Each roadmap step should be runnable/demoable before moving to the next — don't let multiple steps get half-built in parallel.
