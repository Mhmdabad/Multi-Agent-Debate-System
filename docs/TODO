# TODO — HW2: Autonomous Multi-Agent Debate System

Status legend: `[ ]` not started · `[~]` in progress · `[x]` done

---

## Phase 1 — Documentation & Planning

- [ ] Create `docs/PRD.md`
- [ ] Create `docs/PLAN.md` (this file's sibling)
- [ ] Create `docs/TODO.md` (this file)
- [ ] Choose debate topic and define which side each agent takes
- [ ] Review PLAN.md architecture before writing any code

---

## Phase 2 — Project Scaffolding

- [ ] Initialize git repository
- [ ] Create folder structure: `src/`, `tests/`, `docs/`, `config/`, `results/`, `assets/`
- [ ] Create `pyproject.toml` with project metadata and dependencies
- [ ] Run `uv sync` to install dependencies (do NOT use pip)
- [ ] Create `.env-example` with `ANTHROPIC_API_KEY=your_key_here`
- [ ] Create `.env` with real key (git-ignored)
- [ ] Create `.gitignore` (must exclude `.env`, `*.key`, `uv.lock` optional but recommended to commit)
- [ ] Create `src/debate/shared/version.py` with `version = "1.0.0"`
- [ ] Create `config/setup.json` with `max_rounds`, `timeout_seconds`, `model`, `version`
- [ ] Create `config/rate_limits.json` with `requests_per_minute`, `max_retries`, `retry_after_seconds`, `version`
- [ ] Create `src/debate/constants.py` with immutable project constants
- [ ] Commit: "chore: scaffold project structure"

---

## Phase 3 — ApiGatekeeper & Config

- [ ] Implement `src/debate/shared/config.py` — loads `config/*.json` and env vars
- [ ] Implement `src/debate/shared/gatekeeper.py` — `ApiGatekeeper` class
  - [ ] Rate limit check before every call
  - [ ] FIFO queue when limit is reached
  - [ ] Retry logic on transient failure (max_retries from config)
  - [ ] Logging of every API call
- [ ] Write unit tests: `tests/unit/test_gatekeeper.py` (mock the Claude client)
- [ ] Write unit tests: `tests/unit/test_config.py`
- [ ] Verify `ruff check` passes with zero errors
- [ ] Commit: "feat: add ApiGatekeeper and config loader"

---

## Phase 4 — Debater Agent

- [ ] Implement `src/debate/services/debater.py` — `DebaterAgent` class
  - [ ] Accepts: topic, assigned side (FOR / AGAINST), system prompt, skills
  - [ ] Sends arguments via `multiprocessing.Queue`
  - [ ] Calls web search tool on every response
  - [ ] References opponent's previous argument in every response
  - [ ] All Claude calls go through `ApiGatekeeper`
- [ ] Write unit tests: `tests/unit/test_debater.py` (mock Claude API + web search)
- [ ] Test manually via Claude CLI: confirm agent stays on its assigned side
- [ ] Verify `ruff check` passes
- [ ] Commit: "feat: implement DebaterAgent"

---

## Phase 5 — Father Agent (Orchestration)

- [ ] Implement `src/debate/services/father.py` — `FatherAgent` class
  - [ ] Spawns Debater-A and Debater-B as separate processes
  - [ ] Sends each debater its role, topic, and rules before debate starts
  - [ ] Runs the debate loop (reads `max_rounds` from config)
  - [ ] Validates each response — checks agent stayed on-side
  - [ ] If off-side: sends correction and re-asks (does NOT pass bad response to opponent)
  - [ ] Passes each validated response to the other agent
- [ ] Write unit tests: `tests/unit/test_father.py` (mock child processes)
- [ ] Verify `ruff check` passes
- [ ] Commit: "feat: implement FatherAgent debate loop"

---

## Phase 6 — Watchdog

- [ ] Implement `src/debate/services/watchdog.py` — `ProcessWatchdog` class
  - [ ] Runs as daemon thread inside Father process
  - [ ] Polls `.is_alive()` every `WATCHDOG_INTERVAL` seconds (from config)
  - [ ] Detects hung process: time since last response > `TIMEOUT_SECONDS`
  - [ ] On hang: logs warning, terminates process, restarts it with same config
- [ ] Write unit tests: `tests/unit/test_watchdog.py`
- [ ] Verify `ruff check` passes
- [ ] Commit: "feat: implement ProcessWatchdog"

---

## Phase 7 — Final Verdict

- [ ] Extend `FatherAgent` with verdict logic
  - [ ] After all rounds complete, Father reviews full transcript
  - [ ] Produces verdict: winner + specific reasoning referencing actual arguments
  - [ ] Saves verdict to transcript file
- [ ] Write unit tests for verdict generation
- [ ] Commit: "feat: add final verdict to FatherAgent"

---

## Phase 8 — SDK Layer & Entry Point

- [ ] Implement `src/debate/sdk/sdk.py` — `DebateSDK` class
  - [ ] `run(topic: str) -> DebateResult`: single public method that starts everything
  - [ ] All external consumers (main.py) use only this class
  - [ ] No business logic in `main.py`
- [ ] Implement `src/main.py` — reads topic from CLI args, calls `DebateSDK.run()`
- [ ] Write integration test: `tests/integration/test_full_debate.py` (mocked API)
- [ ] Verify `ruff check` passes
- [ ] Commit: "feat: add SDK layer and main entry point"

---

## Phase 9 — Quality Gates

- [ ] Run `uv run pytest tests/ --cov=src --cov-fail-under=85`
- [ ] Fix any coverage gaps until ≥ 85%
- [ ] Run `uv run ruff check src/ tests/` — must show zero errors
- [ ] Check every source file is ≤ 150 lines of code
- [ ] Check all public functions and classes have docstrings
- [ ] Check no hardcoded API keys, URLs, or config values in source code
- [ ] Commit: "chore: fix coverage and linting to meet quality gates"

---

## Phase 10 — End-to-End Run & Submission

- [ ] Run a full debate end-to-end: `uv run python src/main.py`
- [ ] Confirm at least 10 turns per agent (20 total)
- [ ] Confirm each agent stayed on its assigned side throughout
- [ ] Confirm Father intervened at least once (or verify watchdog was exercised)
- [ ] Confirm final verdict was produced with reasoning
- [ ] Save the full transcript to `results/`
- [ ] Take screenshots of the system running
- [ ] Write `README.md`
  - [ ] Installation instructions
  - [ ] How to run
  - [ ] Full transcript (one complete session)
  - [ ] Screenshots
  - [ ] Prompt log (significant prompts used during development)
- [ ] Final commit: "docs: finalize README and transcript for submission"
- [ ] Push to GitHub

---

## Definition of Done (per task)

A task is complete when:
1. Code is written and passes `ruff check`
2. Unit tests exist and pass
3. Coverage for that module is ≥ 85%
4. The file is ≤ 150 lines
5. All public functions have docstrings
