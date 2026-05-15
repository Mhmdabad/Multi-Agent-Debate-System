# PRD — HW2: Autonomous Multi-Agent Debate System

## 1. Overview

Build a Python application that orchestrates three AI agents (one parent, two debaters) to conduct a fully autonomous debate. The system manages process lifecycle, enforces debate rules, and produces a final judgment — all without human input after launch.

---

## 2. Agents

| Agent | Role | Behavior |
|---|---|---|
| **Father (Parent)** | Orchestrator + Judge | Starts everything, enforces rules, monitors agents, gives final verdict |
| **Debater-A** | FOR | Always argues in favor of the chosen topic |
| **Debater-B** | AGAINST | Always argues against the chosen topic |

---

## 3. Functional Requirements

### 3.1 Agent Architecture

| ID | Requirement |
|---|---|
| F-01 | The system shall have exactly three agents: Parent (Judge), Debater-A (FOR), Debater-B (AGAINST) |
| F-02 | Each agent shall run as a separate, independent process |
| F-03 | The Parent shall initialize both debaters, passing each its role, topic, and rules before the debate starts |
| F-04 | Each Debater shall be assigned skills aligned with its one-sided debate position (not neutral/global) |

### 3.2 Debate Flow

| ID | Requirement |
|---|---|
| F-05 | The debate shall consist of at least 10 turns per agent (20 total exchanges, ping-pong style) |
| F-06 | Each agent's response must reference and counter the opponent's previous argument |
| F-07 | Agents must bring concrete examples in their arguments |
| F-08 | Each debating agent must call a web search tool at least once per response (web search is **mandatory**) |
| F-09 | No agent shall agree with the opposing side at any point |
| F-10 | The debate topic is freely chosen by the student; one clear FOR side and one AGAINST side must exist |

### 3.3 Parent / Watchdog Behavior

| ID | Requirement |
|---|---|
| F-11 | The Parent shall monitor all agent outputs continuously throughout the debate |
| F-12 | If a Debater begins agreeing with the opponent, the Parent shall inject a correction forcing it back to its assigned position |
| F-13 | The Parent shall act as a **watchdog**: detect process hangs, kill stalled processes, and restart them automatically |
| F-14 | At the end of all rounds, the Parent shall output a **final verdict**: which debater won, with specific reasoning referencing the actual arguments made |

### 3.4 Skills / Tools

| ID | Requirement |
|---|---|
| F-15 | Each debating agent must have at minimum one web search tool |
| F-16 | Additional skills (e.g., argument generator, argument analyzer, counter-argument builder) are optional but encouraged |
| F-17 | Skills shall be defined at **project scope**, not global scope |
| F-18 | Each debater's skills must reflect its one-sided perspective to maximize contradiction between the two sides |

---

## 4. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NF-01 | The system must run fully autonomously from a single entry point (e.g., `python main.py`) |
| NF-02 | Each process must have a configurable timeout to prevent infinite stalls |
| NF-03 | The watchdog must detect and recover from process failures automatically (kill + restart) |
| NF-05 | The system must handle agent dominance: one agent must not be allowed to "take over" the other |

---

## 5. Project Structure (from submission guidelines)

The project must follow this mandatory folder structure:

```
project-root/
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── sdk/             # SDK layer — single entry point for all logic
│       │   └── sdk.py
│       ├── services/        # Business logic (agents, debate orchestration)
│       ├── shared/
│       │   ├── gatekeeper.py   # API Gatekeeper
│       │   ├── config.py
│       │   └── version.py
│       └── constants.py
│   └── main.py
├── tests/
│   ├── unit/
│   └── integration/
├── docs/
│   ├── PRD.md               # This document
│   ├── PLAN.md              # Architecture & planning
│   └── TODO.md              # Task tracking
├── config/
│   ├── setup.json
│   └── rate_limits.json
├── results/                 # Saved debate transcripts
├── assets/                  # Screenshots, diagrams
├── README.md                # MANDATORY
├── pyproject.toml           # Build & dependencies (NOT requirements.txt)
├── uv.lock                  # Locked dependencies
├── .env-example             # Secret placeholders (committed)
├── .env                     # Actual secrets (git-ignored)
└── .gitignore
```

---

## 6. Code Quality Standards (from submission guidelines)

| Rule | Requirement |
|---|---|
| **File size** | Every source file must stay at or under 150 lines of code (blank lines and comments excluded) |
| **SDK architecture** | All business logic must be accessible through the SDK layer; no logic leaked into CLI/entry-point code |
| **OOP / no duplication** | No duplicated code — if logic appears in 2+ files, extract it to a shared module, base class, or mixin |
| **API Gatekeeper** | All external API calls (to Claude/LLM) must go through a centralized `ApiGatekeeper` class; rate limits read from config, never hardcoded |
| **No hardcoded values** | API URLs, timeouts, rate limits, and all config values must come from `config/` files or environment variables |
| **Docstrings** | Every public function, class, and module must have a docstring explaining WHY, not just WHAT |
| **Naming** | All variable and function names must be descriptive and self-explanatory |
| **Ruff linter** | Code must pass `ruff check` with zero errors |

---

## 7. Testing Standards (from submission guidelines)

| Rule | Requirement |
|---|---|
| **TDD** | Tests must be written before or alongside implementation (Red → Green → Refactor) |
| **Coverage** | Minimum **85% test coverage** globally; the test suite must fail if coverage drops below this |
| **Structure** | `tests/unit/` mirrors `src/` structure; `tests/integration/` covers end-to-end flows |
| **No external calls** | Unit tests must mock all external services (LLM API, web search); no real API calls in unit tests |
| **Edge cases** | Every edge case and error path must have a documented test with expected input/output |

---

## 8. Configuration & Secrets (from submission guidelines)

| Rule | Requirement |
|---|---|
| **No secrets in code** | API keys and tokens must never appear in source code or be committed to Git |
| **Environment variables** | All secrets accessed via `os.environ.get("KEY")` only |
| **`.env-example`** | Must exist with placeholder values showing what variables are required |
| **`.gitignore`** | Must exclude `.env`, `*.key`, `*.pem`, `credentials.json` |
| **Rate limit config** | All rate limits stored in `config/rate_limits.json`, versioned, never hardcoded |

---

## 9. Dependency Management (from submission guidelines)

| Rule | Requirement |
|---|---|
| **`uv` only** | Use `uv` as the sole package manager — `pip install` and `python -m venv` are forbidden |
| **`pyproject.toml`** | Only valid source of truth for dependencies (no `requirements.txt`) |
| **`uv.lock`** | Must exist and be committed; ensures reproducible installs |
| **Running scripts** | All scripts run via `uv run python ...`; all tests via `uv run pytest tests/` |

---

## 10. Submission Requirements

| Item | Details |
|---|---|
| **Code** | Python project on GitHub with a clear `main.py` entry point |
| **`docs/` folder** | Must contain `PRD.md`, `PLAN.md`, `TODO.md` |
| **README** | Full user manual: installation steps, how to run, screenshots, full debate transcript (one complete session) |
| **Transcript** | Markdown or inline in README; English or Hebrew |
| **`.env-example`** | Show which environment variables are needed, without real values |
| **Prompt log** | Document significant prompts used during development (`docs/` or README) |
| **Commit history** | Frequent, meaningful commits showing incremental development (professor checks this) |

---

## 11. Recommended Development Phases

| Phase | Goal |
|---|---|
| 1 | Write `docs/PLAN.md` and `docs/TODO.md`; get architecture approved before coding |
| 2 | Test each agent individually via Claude CLI — confirm it stays in role |
| 3 | Wire the ping-pong loop between the two debaters |
| 4 | Add Parent watchdog + correction logic (force agents back on-role) |
| 5 | Add final verdict logic at the end of the debate |
| 6 | Wrap everything in Python processes with timeouts and watchdog recovery |
| 7 | Add `ApiGatekeeper`, rate-limit config, and `uv` setup |
| 8 | Achieve 85% test coverage; pass `ruff check` with zero errors |
| 9 | Run a full end-to-end session, capture transcript, finalize README |

---

## 12. Key Risk

> The most common failure the professor warned about: **one agent capitulates and starts agreeing** with the other. The Parent's watchdog/correction logic must actively prevent this — it is an explicit grading criterion.

---

## 13. Out of Scope

- GUI or web interface
- Database storage for debate history (file/markdown is sufficient)
- Support for more than two debaters
- Human moderation during the debate
