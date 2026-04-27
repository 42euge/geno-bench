# Getting Started

## Prerequisites

- Python >= 3.10
- A supported coding CLI (Claude Code, Gemini CLI, Codex, or OpenCode)
- [geno-tools](https://github.com/42euge/geno-tools) installed

## Installation

```bash
geno-tools install geno-bench
```

Or from within an agent session:

```
/geno-tools install geno-bench
```

## First use

### 1. List sessions with failure signals

```bash
geno-list-sessions
```

Filter to the most interesting sessions:

```bash
geno-list-sessions --min-thrash 0.3 --min-errors 10
```

### 2. Analyze a specific session

```bash
geno-analyze-session <session_id_prefix>
```

### 3. Write up findings

Create a markdown note in `session_mining/` using the template at `geno_bench/templates/session_note.md`:

```bash
geno-analyze-session 17446aea > session_mining/2026-04-15_my-project_17446aea.md
```

## Python API

```python
from geno_bench import parse_session, discover_sessions, thrashing_score, Session

paths = discover_sessions()
session = parse_session(paths[0])
score, details = thrashing_score(session)
```
