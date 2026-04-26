# geno-bench

Benchmark creation system that mines coding agent session logs for failure patterns and turns them into evaluation tasks.

## Install

```bash
geno-tools install geno-bench
```

Or from within an agent session:

```
/geno-tools install geno-bench
```

## What it does

Coding agents store session logs as JSONL (e.g. `~/.claude/projects/` for Claude Code). Those logs contain every tool call, error, retry, and back-and-forth. geno-bench provides:

1. **Scripts** to scan sessions and surface ones with evidence of agent failure (high thrashing, many errors, long loops)
2. **Analysis** to extract per-session failure patterns (error streaks, repeated resource access, tool distribution)
3. **Templates** for turning mined patterns into benchmark task designs

## Usage

```bash
# List sessions with evidence of failure
geno-list-sessions
geno-list-sessions --min-thrash 0.3 --min-errors 10
geno-list-sessions --project-filter learning

# Deep-dive on a specific session by ID prefix
geno-analyze-session 17446aea

# Pipe output to a markdown note
geno-analyze-session 17446aea > session_notes/2026-04-15_my-project_17446aea.md
```

No third-party dependencies. Uses only the Python standard library.

## Python API

```python
from geno_bench import parse_session, discover_sessions, thrashing_score, Session

paths = discover_sessions()
session = parse_session(paths[0])
score, details = thrashing_score(session)
```

## Package structure

```
geno-bench/
├── pyproject.toml
├── SKILL.md                          # umbrella skill manifest
├── genotools.yaml                    # geno-tools manifest
├── GENO.md                           # agent instructions
├── README.md
├── LICENSE
├── skills/
│   └── geno-bench/SKILL.md           # umbrella skill definition
├── docs/                             # MkDocs Material site
└── geno_bench/
    ├── __init__.py                   # public API
    ├── parser.py                     # JSONL parsing + analysis
    ├── cli/
    │   ├── list_sessions.py          # inventory sessions with failure signals
    │   └── analyze_session.py        # extract patterns from one session
    ├── templates/
    │   ├── session_note.md           # template for per-session mining notes
    │   └── task_catalog.md           # template for failure-to-task catalog
    └── examples/
        └── sample_session_note.md    # what a filled-out note looks like
```

## Philosophy

LLM benchmarks usually test knowledge recall on static datasets. This tool builds benchmarks that test whether models can **learn from observed failures** -- specifically, the kinds of failures that actually occur in agentic coding sessions.

The failure taxonomy that emerges tends to cluster around a handful of cognitive gaps:
- **Stale state recovery** -- retrying with invalidated references
- **Retry without diagnosis** -- hitting the same error repeatedly without updating strategy
- **Tool-task mismatch** -- using a fragile approach when a stable one exists
- **Warning vs error confusion** -- treating non-blocking output as blocking
- **Precondition blindness** -- mutating without checking current state
- **Strategy over-commitment** -- not pivoting when the current approach plateaus

These map naturally to cognitive sub-abilities from learning science and can be instantiated as synthetic benchmark tasks.

## License

MIT
