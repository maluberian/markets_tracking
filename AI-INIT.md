# AI Session Initialization (codex)

## Purpose
This playbook primes an OpenAI codex CLI session with the latest project history and working context. Run these steps immediately after entering the repository root.

## Steps
1. **Review prompt history**
   ```bash
   cat .ai-history/PROMPTS.md
   ```
2. **Load detailed session context**
   ```bash
   cat .ai-history/CONTEXT.md
   ```
3. **Recap current objectives**
   ```bash
   cat NEXT.md
   ```
4. **Open the active PRP**
   ```bash
   cat PRPs/stock_etf_intelligence_harvester.prp.md
   ```
5. **Inspect codex command library (optional refresher)**
   ```bash
   ls .codex/commands
   ```
6. **Acknowledge readiness**
   - Summarize the loaded context back to the user.
   - Confirm the plan of action before making further changes.

Repeat these steps at the start of every codex session to ensure continuity.
