# PLAN — HW2: Autonomous Multi-Agent Debate System

## 1. Architecture Overview

### C4 — Context Level

```
┌─────────────────────────────────────────────────────┐
│                   Student / User                    │
│              runs: uv run python main.py            │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
         ┌─────────────────────────┐
         │   Debate System (app)   │
         │  Python — 3 processes   │
         └──────┬──────────────────┘
                │ calls
                ▼
     ┌──────────────────────┐
     │  Claude API (Anthropic)│
     │  + Web Search Tool    │
     └──────────────────────┘
```

### C4 — Container Level

```
main.py
  │
  └── spawns ──► Father Process (Orchestrator / Judge)
                    │
                    ├── spawns ──► Debater-A Process  (FOR)
                    │                  └── web_search tool
                    │
                    └── spawns ──► Debater-B Process  (AGAINST)
                                       └── web_search tool
```

### C4 — Component Level

```
src/debate/
├── sdk/
│   └── sdk.py              ← DebateSDK: single public API for all operations
├── services/
│   ├── father.py           ← FatherAgent: orchestrates, monitors, judges
│   ├── debater.py          ← DebaterAgent: argues FOR or AGAINST
│   └── watchdog.py         ← ProcessWatchdog: monitors PIDs, restarts on hang
├── shared/
│   ├── gatekeeper.py       ← ApiGatekeeper: rate-limits all Claude API calls
│   ├── config.py           ← loads config/ files + env vars
│   └── version.py          ← version = "1.0.0"
└── constants.py            ← immutable project-wide constants
```

---

## 2. Data Flow

```
1. main.py calls DebateSDK.run(topic)
2. DebateSDK starts Father process
3. Father sends each Debater its system prompt (role + topic + rules)
4. Father starts the debate loop (max_rounds from config):
     a. Ask Debater-A to respond
     b. Father validates response — is it on-side?
        → if NOT: Father sends correction, re-asks
     c. Pass Debater-A's argument to Debater-B
     d. Ask Debater-B to respond
     e. Father validates — same check
     f. Repeat until max_rounds reached
5. Father produces final verdict with reasoning
6. Transcript written to results/debate_<timestamp>.md
```

---

## 3. Process Communication

Each agent runs as a **separate Python process** (via `multiprocessing.Process`).  
Communication between processes uses `multiprocessing.Queue` (FIFO, thread-safe):

```
Father ──[Queue]──► Debater-A  (sends prompt/correction)
Father ◄──[Queue]── Debater-A  (receives response)

Father ──[Queue]──► Debater-B
Father ◄──[Queue]── Debater-B
```

The Watchdog runs in the Father process as a background thread, polling process `.is_alive()` every N seconds (configurable).

---

## 4. ApiGatekeeper Design

All calls to the Claude API go through a single `ApiGatekeeper` instance:

```python
class ApiGatekeeper:
    """Routes all LLM calls through rate-limit enforcement and retry logic."""

    def execute(self, api_call, *args, **kwargs):
        # 1. Check rate limits (from config/rate_limits.json)
        # 2. Queue if limit reached (FIFO queue)
        # 3. Execute call
        # 4. Retry on transient failure (max_retries from config)
        # 5. Log call metadata
        ...
```

Rate limits are read from `config/rate_limits.json` — never hardcoded.

---

## 5. Configuration Architecture

```
config/
├── setup.json          ← debate settings: max_rounds, timeout_seconds, model name
└── rate_limits.json    ← requests_per_minute, max_retries, retry_after_seconds

.env                    ← ANTHROPIC_API_KEY (git-ignored)
.env-example            ← ANTHROPIC_API_KEY=your_key_here
```

All config values accessed via `src/debate/shared/config.py` — never read directly in business logic.

---

## 6. Watchdog Design

The watchdog runs as a daemon thread inside the Father process:

```
every WATCHDOG_INTERVAL seconds:
  for each child process (Debater-A, Debater-B):
    if not process.is_alive():
      log warning
      restart process with same config
    elif time_since_last_response > TIMEOUT_SECONDS:
      log warning
      terminate process
      restart process
```

Both `WATCHDOG_INTERVAL` and `TIMEOUT_SECONDS` are read from `config/setup.json`.

---

## 7. Architectural Decisions (ADRs)

### ADR-01: Multiprocessing over Multithreading
- **Decision**: Use `multiprocessing.Process` for each agent.
- **Reason**: Each agent makes blocking I/O calls to the Claude API. Separate processes give true isolation — a hung agent cannot block the Father. Also avoids Python's GIL for CPU-side work.
- **Trade-off**: Higher memory overhead vs. threads; accepted because agent count is fixed at 3.

### ADR-02: Queue-based IPC over shared memory
- **Decision**: Use `multiprocessing.Queue` for inter-process communication.
- **Reason**: Simple, safe, no shared-state race conditions. Messages are strings (prompts/responses).
- **Trade-off**: Slightly higher latency than shared memory; irrelevant at debate timescales.

### ADR-03: Father validates every response before passing it on
- **Decision**: The Father checks each response for off-role behavior before the next agent sees it.
- **Reason**: Prevents one capitulating agent from infecting the other. Explicit grading criterion.
- **Trade-off**: Adds one extra LLM call per turn for validation; accepted for correctness.

### ADR-04: Skills are project-scoped, not global
- **Decision**: All Claude tools/skills defined in `config/` or code, not in the global Claude CLI profile.
- **Reason**: Prevents skill bleed into unrelated sessions; required by the professor.

---

## 8. Versioning

| Item | Location | Initial value |
|---|---|---|
| Code version | `src/debate/shared/version.py` | `1.0.0` |
| Config version | `config/setup.json` → `"version"` | `1.0.0` |
| Rate limits version | `config/rate_limits.json` → `"version"` | `1.0.0` |

The app validates that config version matches code version on startup.
