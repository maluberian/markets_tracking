name: "Stock & ETF Intelligence Harvester MVP - codex Base PRP"
description: |
  Implementation-focused PRP for OpenAI codex CLI to ship the first iteration of a
  value-investing oriented market data harvester using free-tier public APIs with
  <=15 minute delayed quotes.

---

## Goal

**Feature Goal**: Deliver a Python workflow that gathers valuation fundamentals and quote data for a user-supplied list of US stocks and ETFs, returning normalized JSON snapshots ready for downstream analysis.

**Deliverable**: Executable CLI `python -m src.cli.fetch --tickers <CSV>` with modular clients, normalization pipeline, caching, and automated tests backed by recorded fixtures.

**Success Definition**: Running the CLI for a sample list of >=5 tickers yields JSON artifacts under `data/out/` (or stdout) containing quote + fundamentals, respects API rate limits, and passes all validation gates without manual corrections.

## User Persona (if applicable)

**Target User**: Solo retail value investor automating research workflows (the repo owner).

**Use Case**: Trigger ad-hoc or scheduled fetches to compare valuation metrics (P/E, EV/EBITDA, dividend yield, free cash flow per share) across candidate equities and ETFs.

**User Journey**:
1. Configure API keys in `.env` (Alpha Vantage required, FMP optional fallback).
2. Provide tickers via CSV or comma-separated string.
3. Execute CLI with `--dry-run` to confirm plan, then with live fetch to cache and emit normalized JSON.
4. Inspect outputs in `data/out/<timestamp>/` or pipe to downstream tooling.

**Pain Points Addressed**:
- Manual copy/paste of fundamentals from multiple sites.
- Lack of reproducible dataset for screening models.
- Desire for low-cost (<$100) setup without premium market data feeds.

## Why

- Enables consistent, repeatable collection of fundamentals for value-investing decisions.
- Establishes a foundation for future analytics, storage, and strategy automation.
- Relies on free-tier public APIs to stay within the initial budget while providing useful metrics.

## What

- CLI orchestrates ticker normalization, API retrieval (primary Alpha Vantage, fallback Financial Modeling Prep), normalization into typed Pydantic models, and JSON serialization.
- Handles both equity and ETF tickers, surfacing actionable errors for unsupported symbols or quota issues.
- Provides dry-run mode to preview planned API calls without network requests.
- Caches raw responses to avoid breaching rate limits, with configurable cache directory.
- Outputs machine-friendly JSON (array or newline-delimited) including provenance metadata.

### Success Criteria

- [ ] `codex` creates the module layout described in the desired codebase tree.
- [ ] Running `python -m src.cli.fetch --tickers examples/sample_tickers.csv` generates JSON output under `data/out/`.
- [ ] CLI supports arguments: `--tickers`, `--output`, `--as-of`, `--dry-run`, `--format` (`json` or `ndjson`).
- [ ] Tests simulate API responses via fixtures and pass with `pytest`.
- [ ] Validation loop commands complete without manual fixes.
- [ ] Updated README (or docs/setup.md) provides <1 page setup instructions oriented to codex users.

## All Needed Context

### Context Completeness Check
Affirm that an engineer unfamiliar with the repo can implement the feature using only this PRP, referenced docs, and the existing codebase.

### Documentation & References

```yaml
- url: https://www.alphavantage.co/documentation/
  why: Primary free-tier API for quotes, fundamentals, and ETF endpoints.
  critical: Respect 5 calls/min & 500 calls/day; equities vs ETF functions differ; parse Retry-After headers.

- url: https://site.financialmodelingprep.com/developer/docs
  why: Secondary fundamentals source with generous free tier; offers key metrics, ETF profiles, cash flow data.
  critical: Append `apikey` query param; free tier limited to 250 calls/day; HTTP 402 indicates quota exhaustion.

- url: https://python-httpx.org/quickstart/
  why: Preferred async HTTP client for rate-limited APIs; built-in timeouts and retry strategy.
  critical: Demonstrates client/session usage patterns relevant for throttled fetches.

- file: README.md
  why: Keep repository overview and quickstart instructions aligned with new CLI workflow.
  pattern: Document environment setup, command usage, and validation loop.

- docfile: PRPs/ai_docs/codex_best_practices.md
  why: Contains agentic execution best practices to adapt for codex CLI.
  section: Validation & self-check patterns (translate "Claude" references to "codex").
```

### Current Codebase tree

```text
.
|- PRP.md
|- README.md
|- PRPs/
|  |- README.md
|  |- templates/
|  |- scripts/
|  `- ai_docs/
`- .codex/
   `- commands/
```

### Desired Codebase tree with files to be added and responsibility of file

```text
.
|- .codex/
|  `- commands/              # codex command prompts (renamed from Claude assets)
|- PRPs/
|  |- README.md
|  |- templates/
|  |- scripts/
|  |- ai_docs/
|  `- stock_etf_intelligence_harvester.prp.md
|- README.md                 # Updated with codex workflow + CLI instructions
|- PRP.md                    # (Optional) pointer to active PRP
|- pyproject.toml            # Project metadata + dependencies (httpx, pydantic, typer, python-dotenv, respx, pytest, rich)
|- .env.example              # Document ALPHAVANTAGE_API_KEY, FMP_API_KEY (optional), CACHE_DIR
|- .gitignore                # Ensure .env, data/out, .cache excluded
|- data/
|  `- out/                   # Generated JSON outputs (gitignored)
|- examples/
|  `- sample_tickers.csv     # Smoke-test tickers
|- src/
|  |- __init__.py
|  |- config.py              # Pydantic settings loader for env vars and defaults
|  |- utils/
|  |  `- ticker_normalization.py  # Shared symbol normalization helpers
|  |- clients/
|  |  |- __init__.py
|  |  |- alpha_vantage.py    # API client with throttling + endpoints for quotes/fundamentals/ETF data
|  |  `- fmp.py              # Backup API client for fundamentals
|  |- models/
|  |  |- __init__.py
|  |  `- asset_snapshot.py   # Pydantic models for normalized outputs + raw schema fragments
|  |- pipelines/
|  |  `- snapshot_builder.py # Merge raw responses into AssetSnapshot with missing-field handling
|  |- services/
|  |  |- __init__.py
|  |  `- ingest_service.py   # High-level orchestration, caching, fallback logic
|  |- storage/
|  |  `- json_writer.py      # Serialization to json/ndjson with timestamped directories
|  `- cli/
|     |- __init__.py
|     `- fetch.py            # Typer CLI entry point invoking ingest service
`- tests/
   |- conftest.py
   |- fixtures/
   |  |- alpha_vantage_overview.json
   |  |- alpha_vantage_time_series.json
   |  `- fmp_key_metrics.json
   |- test_alpha_vantage_client.py
   |- test_fmp_client.py
   |- test_ingest_service.py
   `- test_cli_fetch.py
```

### Known Gotchas of our codebase & Library Quirks

```python
# Alpha Vantage free tier allows 5 requests/minute -> implement per-request delay or semaphore.
# ETF endpoints differ from equity endpoints (e.g., ETF_PROFILE); missing metrics should map to None, not crash.
# Financial Modeling Prep returns HTTP 402 when quota exceeded -> treat as retryable with exponential backoff and fallback to cached data.
# Symbols with dots or slashes (BRK.B, RDS/A) need consistent normalization before hitting providers.
# Cache raw responses under .cache/ keyed by symbol + function + date to avoid duplicate calls in one run.
# Tests must mock HTTP requests (use respx) because network access may be restricted during CI or codex runs.
```

## Implementation Blueprint

### Data models and structure

- `AssetSnapshot` (Pydantic BaseModel) capturing symbol metadata, price quote, valuation metrics, updated_at timestamp, and data_source provenance.
- `Quote` model with price, volume, open/high/low, 52-week high/low, last refreshed.
- `Fundamentals` model with valuation ratios (pe_ratio, pb_ratio, ev_to_ebitda, dividend_yield, payout_ratio, free_cash_flow_per_share, nav for ETFs).
- `FetchPlan` data class capturing requested tickers, as-of date, and chosen data providers for logging/dry-run output.

### Implementation Tasks (ordered by dependencies)

```yaml
Task 1: CREATE pyproject.toml & base tooling
  - DEFINE project metadata (Python >=3.11) and add dependencies: httpx, pydantic, python-dotenv, typer, respx, pytest, rich.
  - ADD optional dev dependencies: ruff, mypy, pytest-cov.
  - UPDATE README.md with codex-specific setup and validation loop summary.

Task 2: CREATE src/config.py
  - IMPLEMENT Pydantic BaseSettings class with fields for API keys, cache directory, request limits.
  - INCLUDE helper `get_settings()` with lru_cache to avoid repeated env parsing.
  - PROVIDE friendly error messages when mandatory API key missing for live fetches.

Task 3: CREATE utility helpers
  - ADD `src/utils/ticker_normalization.py` to standardize ticker casing, convert "BRK.B" to provider-specific forms, and strip whitespace.
  - INCLUDE test coverage for edge cases.

Task 4: CREATE API clients
  - IMPLEMENT `AlphaVantageClient` using httpx.AsyncClient with rate limiting, `get_overview`, `get_time_series_daily`, `get_etf_profile` methods.
  - HANDLE throttling via asyncio.Semaphore or sleep, parse Retry-After headers, and raise custom exceptions (ExternalAPIError, QuotaExceededError).
  - IMPLEMENT `FMPClient` with matching interface for fundamentals fallback.
  - CENTRALIZE base URLs, query param building, and response validation.

Task 5: CREATE models and normalization pipeline
  - DEFINE Pydantic models under `src/models/asset_snapshot.py` for raw + normalized shapes.
  - IMPLEMENT `src/pipelines/snapshot_builder.py` to merge responses into AssetSnapshot, logging missing metrics and defaulting gracefully.

Task 6: CREATE ingest service
  - ADD `IngestService` with `fetch_snapshots(tickers, as_of, dry_run, use_cache)` method orchestrating normalization, caching, fallback between providers.
  - SUPPORT optional caching in `.cache/` with TTL per as-of date.
  - RETURN detailed fetch report for CLI summary.

Task 7: CREATE storage layer
  - IMPLEMENT `json_writer.write_snapshots(snapshots, output_path, format)` to write JSON array or NDJSON, creating timestamped directories when needed.
  - HANDLE stdout streaming when `--output -` provided.

Task 8: CREATE CLI entry point
  - USE Typer to parse options (`--tickers`, `--output`, `--as-of`, `--dry-run`, `--format`, `--use-cache/--no-cache`).
  - SHOW dry-run preview of provider calls, skip network when flag set.
  - AFTER fetch, print summary table using rich (symbol, data_source, updated_at, key ratios).

Task 9: SETUP tests and fixtures
  - RECORD sample API responses (redact API keys) under `tests/fixtures/`.
  - USE respx to mock httpx for positive path, rate limit, and failure scenarios.
  - COVER CLI invocation with Typer testing utilities.

Task 10: ENHANCE developer experience
  - ADD `.env.example`, update `.gitignore` for `.env`, `.cache/`, `data/out/`.
  - DOCUMENT makefile/uv commands (optional) and validation loop in README.
```

### Non-Functional Requirements

- Execution for 10 tickers completes under 2 minutes on free-tier quotas.
- Logging includes timestamps + severity; CLI prints concise summary while writing verbose logs to file if configured.
- Code designed for extensibility (e.g., additional data providers implementing same interface).
- Budget-friendly: rely on free tiers; clearly gate optional paid upgrades.

### Agent Execution Guidance (codex-specific)

- Always start by reading this PRP and `PRPs/README.md` to internalize methodology.
- Replace any residual references to `.claude` commands with `.codex` equivalents when planning codex runs.
- Use plan-and-execute: enumerate tasks, run validations after each major milestone, and avoid skipping tests.
- Avoid network calls during automated validation; rely on recorded fixtures unless performing manual smoke tests.

## Validation Loop

### Level 0: Environment Setup (manual once)

```bash
uv venv --python 3.11
uv pip install -e .
cp .env.example .env  # populate API keys before live fetch
```

### Level 1: Static Analysis & Formatting

```bash
ruff check src tests
ruff format --check src tests
mypy src
```

### Level 2: Unit Tests

```bash
pytest tests -v
```

### Level 3: Integration Smoke Test (manual / optional live data)

```bash
python -m src.cli.fetch --tickers examples/sample_tickers.csv --dry-run
python -m src.cli.fetch --tickers "AAPL,MSFT,SPY" --output data/out/latest.json
jq '.[0] | {symbol, pe_ratio, dividend_yield, data_source}' data/out/latest.json
```

### Level 4: Quality Gates

```bash
python -m src.cli.fetch --tickers "AAPL" --dry-run  # Confirm no network usage
pytest --cov=src --cov-report=term-missing
```

## Final Validation Checklist

### Technical Validation

- [ ] All validation loop commands succeed locally.
- [ ] No hardcoded API keys; configuration strictly via env vars/settings module.
- [ ] Caching avoids duplicate network calls within same run.
- [ ] JSON output validates against `AssetSnapshot` schema.

### Feature Validation

- [ ] CLI supports equities and ETFs with clear provenance metadata.
- [ ] Dry-run mode accurately previews planned API calls.
- [ ] Errors for quota exhaustion or unsupported symbols include remediation tips.

### Documentation & Developer Experience

- [ ] README (or docs/setup.md) updated with codex workflow, environment setup, validation loop.
- [ ] `.env.example` lists required and optional env vars with descriptions.
- [ ] PRP moved to `PRPs/completed/` after implementation (per methodology).

## Anti-Patterns to Avoid

- X Skipping caching or rate limiting (will violate free-tier quotas).
- X Adding new patterns instead of following provided module layout.
- X Making live API calls inside automated tests; always stub responses.
- X Hardcoding ticker-specific logic without using normalization helpers.

## Future Enhancements & Open Questions

- Schedule recurring fetches via cron or workflow automation once MVP validated.
- Expand providers (Tiingo, Twelve Data) when budget permits and document switching strategy.
- Persist normalized data to SQLite/PostgreSQL or cloud storage.
- Compute derived quality metrics (Piotroski F-score, dividend growth) locally using fetched data.
- Open question: preferred hosting or deployment environment for long-running ingest service?

---

Use this PRP as the definitive runbook when instructing codex to implement the MVP ingestion workflow. Update as constraints evolve.
