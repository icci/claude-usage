# Usage Guide

All commands use `python3` on macOS/Linux. Windows users can use `python` instead. No virtual environment or `pip install` needed — this is pure stdlib Python.

## Commands

### `scan` — Build or update the database

```bash
python3 cli.py scan
```

Reads Claude Code JSONL transcripts from `~/.claude/projects/` and writes aggregated data to `~/.claude/usage.db`. The scanner is incremental — it tracks file paths and modification times, so re-runs only process new or changed files.

Example output:

```
Scanning /Users/you/.claude/projects ...
  New files:     854
  Updated files: 0
  Skipped files: 0
  Turns added:   80609
  Sessions seen: 182
```

### `today` — Terminal summary for today

```bash
python3 cli.py today
```

Shows token usage and estimated cost for the current day, broken down by model.

Example output:

```
------------------------------------------------------------
  Today's Usage  (2026-04-08)
------------------------------------------------------------
  claude-opus-4-6                 turns=418   in=50.5K     out=94.5K     cost=$66.4843
  claude-haiku-4-5-20251001       turns=64    in=2.1K      out=7.1K      cost=$0.4866
  claude-sonnet-4-6               turns=43    in=169       out=4.3K      cost=$0.6415
------------------------------------------------------------
  TOTAL                           turns=525   in=52.7K     out=105.9K    cost=$67.6124

  Sessions today:   4
  Cache read:       24.69M
  Cache creation:   1.76M
------------------------------------------------------------
```

### `stats` — All-time terminal statistics

```bash
python3 cli.py stats
```

Shows lifetime totals, per-model breakdown, top projects, and 30-day daily averages.

Example output:

```
============================================================
  Claude Code Usage - All-Time Statistics
============================================================
  Period:           2026-02-25 to 2026-04-08
  Total sessions:   182
  Total turns:      80.7K

  Input tokens:     3.23M         (raw prompt tokens)
  Output tokens:    22.46M        (generated tokens)
  Cache read:       13762.80M     (90% cheaper than input)
  Cache creation:   342.40M       (25% premium on input)

  Est. total cost:  $25158.8132
------------------------------------------------------------
  By Model:
    claude-opus-4-6                 sessions=111   turns=69.0K   ...
    claude-haiku-4-5-20251001       sessions=23    turns=11.1K   ...
    claude-sonnet-4-6               sessions=4     turns=638     ...
------------------------------------------------------------
  Top Projects:
    Users/aaron                               sessions=147  turns=73.4K   tokens=23.56M
    GitHub/icci-workspace-mcp                 sessions=22   turns=4.3K    tokens=1.20M
    ...
============================================================
```

### `dashboard` — Web dashboard with charts

```bash
python3 cli.py dashboard
```

Runs `scan` first, then starts an HTTP server on `localhost:8080` and opens your browser. The dashboard auto-refreshes every 30 seconds.

Features:
- Stacked bar chart of daily token usage (input, output, cache read, cache creation)
- Doughnut chart of usage by model
- Horizontal bar chart of top projects by tokens
- Session table with per-session cost estimates
- Model cost breakdown table
- Filter by model (checkboxes) and time range (7d / 30d / 90d / all)
- Bookmarkable URLs — filter state is persisted in query parameters

To stop the server, press `Ctrl+C`.

## Common Workflows

**Quick morning check:**

```bash
cd ~/Documents/GitHub/claude-usage
python3 cli.py scan && python3 cli.py today
```

**Full dashboard session:**

```bash
cd ~/Documents/GitHub/claude-usage
python3 cli.py dashboard
```

**Just update the database (useful before querying `usage.db` directly):**

```bash
python3 cli.py scan
```

## Database Location

The SQLite database is stored at `~/.claude/usage.db`. You can query it directly if you want custom reports:

```bash
sqlite3 ~/.claude/usage.db "SELECT model, COUNT(*) as turns, SUM(input_tokens) as input_tok FROM turns GROUP BY model ORDER BY input_tok DESC;"
```

## Troubleshooting

**"Database not found" error:** Run `python3 cli.py scan` first to build the database from your Claude Code transcripts.

**No data showing up:** Claude Code must have been used at least once. Check that `~/.claude/projects/` contains JSONL files.

**Port 8080 already in use:** Kill the existing process (`lsof -i :8080`) or modify the port in `dashboard.py`'s `serve()` function.

**Cowork sessions missing:** Cowork sessions run server-side and do not write local JSONL transcripts. They cannot be tracked by this tool.
