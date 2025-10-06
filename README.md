# Stock & ETF Intelligence Harvester (codex Workspace)

## Mission
Establish a codex-driven workflow for building a value-investing market data harvester that pulls fundamentals and quotes for US stocks and ETFs via free-tier APIs. The Product-Requirements-Prompt (PRP) in `PRPs/stock_etf_intelligence_harvester.prp.md` is the single source of truth for scope, architecture, validation, and future enhancements.

## Session Boot Sequence (run this at the start of every codex session)
1. `cat AI-INIT.md` and execute each step to load `.ai-history/` context, `NEXT.md`, and the active PRP.
2. Summarize the refreshed context back to the operator before taking action.
3. Plan the upcoming work in alignment with the PRP implementation tasks and `NEXT.md` priorities.

## Core Artifacts
- `.codex/commands/` -- codex-ready command prompts adapted from the original Claude framework.
- `PRPs/` -- templates, scripts, AI documentation, and the active PRP runbook.
- `.ai-history/` -- session memory (`PROMPTS.md`, `CONTEXT.md`) for continuity across codex runs.
- `NEXT.md` -- highlights, required files, and immediate actions before coding begins.
- `AI-INIT.md` -- standardized initialization checklist.

## Current Implementation Status
- Project is in planning/setup phase; application code, configuration files, and tests are not yet created.
- Required scaffolding (see desired tree inside the PRP) still needs to be added: `pyproject.toml`, `.env.example`, `src/` modules, test suite, fixtures, and data directories.
- API keys (Alpha Vantage mandatory, Financial Modeling Prep optional) must be secured before live fetches.

## Next Work Items (see `NEXT.md` for detail)
- Scaffold project structure and configuration.
- Implement API clients, normalization pipeline, ingest service, storage writer, and Typer CLI entrypoint.
- Create fixtures and tests using respx mocks; avoid live network calls during validation.
- Execute the validation loop defined in the PRP (ruff, mypy, pytest, CLI dry-run) once modules are in place.

## Validation References
When implementation begins, the validation commands listed under **Validation Loop** in `PRPs/stock_etf_intelligence_harvester.prp.md` are mandatory before declaring work complete.

## Communication Guidance
- Treat `PRPs/stock_etf_intelligence_harvester.prp.md` as the authoritative spec.
- Document new findings or adjustments by updating the PRP and `.ai-history/` files so future sessions remain synchronized.
