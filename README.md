# Enterprise AI Agent & Investigation Platform

This project explores how an investigation workflow can use retrieved evidence while keeping decisions and approvals traceable. The example is a financial dispute: an analyst opens a case, checks the supporting records, prepares a review packet, and hands the final decision to someone with reviewer access.

Everything runs on synthetic cases. It doesn't connect to payment systems or make real dispute decisions.

## What is implemented

- A FastAPI service and browser-based investigation console.
- Case records in SQLite, including revisions, evidence, review state, idempotency results, and an append-only audit history.
- Tenant and role checks on API requests. A separate reviewer credential is required to approve or reject a case.
- Local evidence retrieval using TF-IDF and lexical matching. Results include excerpts and source IDs so the analyst can inspect the basis for a recommendation.
- Checks for missing evidence and suspicious instructions embedded in retrieved text.
- A deterministic investigation pipeline. An optional LangGraph adapter runs its steps as a graph; neither path needs a paid model API.
- Read-only, tenant-scoped MCP tools for local development.

The deliberate boundary here is that evidence retrieval can help prepare a case, but it cannot approve one on its own.

## Run it locally

Use Python 3.11 or newer.

```bash
python -m pip install -r requirements.txt
DEMO_MODE=1 uvicorn backend.api:app --reload --port 8000
```

Open http://127.0.0.1:8000. The interface includes credentials for the local demo. Do not expose `DEMO_MODE=1` on a public server.

To populate example investigations:

```bash
DEMO_MODE=1 PYTHONPATH=. python -m scripts.seed_demo
```

The sample cases cover a normal dispute, a case with missing evidence, and a document containing an attempted prompt injection.

Or run the app in Docker:

```bash
docker build -t enterprise-agent-platform .
docker run --rm -p 127.0.0.1:8000:8000 -e DEMO_MODE=1 -v investigation-data:/srv/runtime enterprise-agent-platform
```

## Tests

```bash
PYTHONPATH=. pytest -q
PYTHONPATH=. python -m eval.run_eval
```

For the optional graph and local MCP interface:

```bash
python -m pip install 'langgraph>=0.4,<2' 'mcp>=1.8,<2'
DEMO_MODE=1 python -m backend.mcp_tools
```

The MCP interface uses stdio and provides read-only tools; it is not a remotely hosted MCP server.

## Current limits

This is a local engineering prototype. There is no LLM-based reasoning, production SSO, durable LangGraph checkpointing, managed vector database, or remote MCP security layer. The SQLite case store persists records, but that is different from persisting an agent's execution state. Before using this design with actual customer information, it would need a security review, managed identity, appropriate encryption/key management, and more thorough authorization and failure-mode testing.

[CASE_STUDY.md](CASE_STUDY.md) walks through the design choices and evaluation. [SECURITY.md](SECURITY.md) lists the security boundaries in more detail.
