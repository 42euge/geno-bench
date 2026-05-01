# geno-bench -- benchmark creation system for coding agent evaluation

Mines coding agent session logs (JSONL) for failure patterns -- error loops, thrashing, retry-without-diagnosis -- and abstracts them into synthetic evaluation tasks grounded in real agent failures.

## Skills

| Skill | Sub-skillset | Slash command |
|-------|-------------|---------------|
| geno-bench | -- | -- (umbrella) |

## Repo structure

```
geno-bench/
├── GENO.md              # agent instructions (this file)
├── SKILL.md             # umbrella skill manifest
├── genotools.yaml       # geno-tools manifest
├── pyproject.toml       # Python package
├── README.md            # human-readable documentation
├── LICENSE              # MIT
├── skills/
│   └── geno-bench/      # umbrella skill
│       └── SKILL.md
├── docs/                # MkDocs Material site
│   ├── index.md
│   └── getting-started.md
├── mkdocs.yml           # docs site config
└── geno_bench/          # Python package
    ├── __init__.py      # public API (Session, parse_session, discover_sessions, thrashing_score)
    ├── parser.py        # JSONL parsing + analysis (stdlib only, no deps)
    ├── cli/
    │   ├── list_sessions.py     # geno-list-sessions CLI
    │   └── analyze_session.py   # geno-analyze-session CLI
    ├── templates/
    │   ├── session_note.md      # per-session mining note template
    │   └── task_catalog.md      # failure-to-task catalog template
    └── examples/
        └── sample_session_note.md
```

## Conventions

- **Skill naming**: skills follow the `geno-bench-{sub-skillset}-{skill}` pattern per the geno-tools nomenclature spec.
- **Prefix aliasing**: the canonical slash command prefix is `/geno-bench-*`. The short alias `/gt-bench-*` is registered for convenience. Both forms resolve to the same skill. Aliases are declared in each skill's `SKILL.md` front-matter under `aliases:`.
- **Adding a new skill**: create `skills/<sub-skillset>/SKILL.md` with the required front-matter (`name`, `description`, `aliases`). Add a row to the Skills table above. If the skill needs Python, add an entry point in `pyproject.toml` under `[project.scripts]`.
- **No third-party dependencies**: the Python package uses only the standard library.
- **Session data privacy**: never commit raw session data to the repo. Mining notes reference session IDs and counts, not raw content.

## Architecture

### Session parsing

`geno_bench.parser` reads JSONL session logs (one JSON record per line) and builds `Session` objects containing structured `Turn` and `ToolCall` records. It links `tool_result` blocks back to their `tool_use` parents by ID.

### Analysis signals

The parser extracts five signals:
1. **Error patterns** -- tool calls with failing results, grouped by type
2. **Error streaks** -- 3+ consecutive failures (no-progress loops)
3. **Thrashing** -- same resource accessed 3+ times (`thrashing_score()`)
4. **Tool distribution** -- which tools dominate
5. **Strategy commitment** -- over-commitment to single approach

### CLI entry points

- `geno-list-sessions` -- scans agent session log directories (e.g. `~/.claude/projects/**/*.jsonl` for Claude Code; other agents may store logs elsewhere), surfaces sessions with high thrashing/errors
- `geno-analyze-session <id>` -- deep-dive on a single session, outputs markdown-formatted analysis

### Python API

```python
from geno_bench import parse_session, discover_sessions, thrashing_score, Session

paths = discover_sessions()
session = parse_session(paths[0])
score, details = thrashing_score(session)
```

## Dependencies and runtime

- Python >= 3.10
- No third-party dependencies (stdlib only)
- Requires a venv created by `geno-tools install`
