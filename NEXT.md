# NEXT

## Highlights
- `.codex/commands/development/prime-core.md:1` and related prompts now speak in codex terminology so imported command workflows align with the OpenAI codex CLI.
- `PRPs/stock_etf_intelligence_harvester.prp.md:1` holds the end-to-end base PRP covering context, architecture, validation gates, and future work for the stock/ETF data harvester MVP.
- `PRP.md:1` and `README.md:1` both point directly to the codex-centric workflow, making the active PRP and supporting assets easy to find.
- `PRPs/scripts/prp_runner.py:1` defaults to codex when executing PRPs, matching the new command tooling.
- `PRPs/ai_docs/codex_adapter_notes.md:1` records how to interpret the legacy docs while everything transitions to codex specifics.
- Updated every template in `PRPs/templates/` with codex guidance, ASCII-safe content, and notes to use the prompts under `.codex/commands/`.
- Renamed the Claude-oriented AI docs to `codex_*.md`, converted their contents to codex terminology, and scrubbed non-ASCII glyphs for consistency.

## Files to Prepare for Implementation
- `pyproject.toml` - define project metadata and dependencies (`httpx`, `pydantic`, `typer`, `python-dotenv`, `respx`, `pytest`, `rich`, plus dev tools like `ruff` and `mypy`).
- `.env.example` - document `ALPHAVANTAGE_API_KEY`, optional `FMP_API_KEY`, cache directory defaults, and any rate-limit overrides.
- `.gitignore` - exclude `.env`, `.cache/`, `data/out/`, `__pycache__/`, and other generated artifacts.
- `data/out/` & `.cache/` placeholders - ensure directories exist (or are created on demand) and remain ignored.
- `examples/sample_tickers.csv` - provide a starter ticker list for smoke tests (`AAPL,MSFT,SPY,BRK.B,VTI`, etc.).
- `src/config.py` - load and validate settings via `pydantic.BaseSettings` with helpful error messages.
- `src/utils/ticker_normalization.py` - normalize symbols across Alpha Vantage and FMP quirks.
- `src/clients/alpha_vantage.py` and `src/clients/fmp.py` - async HTTPX clients with throttle handling, error mapping, and retry/backoff logic.
- `src/models/asset_snapshot.py` - Pydantic models for quotes, fundamentals, and the composite snapshot payload.
- `src/pipelines/snapshot_builder.py` - transform raw API payloads into `AssetSnapshot` instances with provenance tracking.
- `src/services/ingest_service.py` - orchestrate fetch plans, caching, provider fallback, and summary reporting.
- `src/storage/json_writer.py` - write JSON array or NDJSON outputs, handle stdout mode, and create timestamped folders.
- `src/cli/fetch.py` - Typer CLI entrypoint exposing `--tickers`, `--output`, `--as-of`, `--dry-run`, and cache toggles.
- `tests/` suite (`conftest.py`, fixture JSON files, client/service/CLI tests) - enforce mocked HTTP interactions and validation coverage.
- `README.md` (or `docs/setup.md`) - expand quickstart instructions to cover environment setup, codex usage, and the validation loop.

## Immediate Next Actions
1. Secure free-tier API keys (Alpha Vantage required, FMP optional) and populate `.env` after creating `.env.example`.
2. Stand up the project skeleton from the desired tree, beginning with `pyproject.toml`, `src/` package scaffolding, and the Typer CLI stub.
3. Implement the HTTP client layer with rate limiting, retries, and recorded fixture capture for tests.
4. Build out normalization/pipeline modules, then wire up the ingest service, storage writer, and CLI entrypoint.
5. Generate test fixtures (using manual fetches) and write `pytest` suites leveraging `respx` mocks to keep validation offline-friendly.
6. Run the validation loop from the PRP (ruff, mypy, pytest, dry-run CLI) to confirm the MVP is ready for live data pulls.
