# SPD — Systematic Problem Decomposer

An agent skill that makes an AI coding agent **classify, decompose and solve complex tasks in a fixed order** instead of jumping straight into edits.

SPD combines five well-known methods into one protocol:

| Phase | Method | What it does |
| :--- | :--- | :--- |
| 1 | **Cynefin triage** | Classifies the task (Clear / Complicated / Complex / Chaotic) before any mutation |
| 2 | **MECE issue tree** | Maps the problem space with non-overlapping, exhaustive branches |
| 3 | **Theory of Constraints** | Picks the single bottleneck or most likely root cause; defers everything else |
| 4 | **Hierarchical Task Analysis** | Writes a numbered plan with a verification gate per step |
| 5 | **Checkpoint & iterate** | Summarises verified state, then repeats or concludes |

A **circuit breaker** (First Principles reset) halts the agent after two consecutive failed steps, forces it to list and test its assumptions, and rebuilds the plan from verified facts only.

The full protocol is in [`SKILL.md`](SKILL.md).

## Usage

Tree depth is set with an optional argument. The default is `depth=2`.

| Depth | Alias | Scope | Best for |
| :--- | :--- | :--- | :--- |
| `depth=1` | `shallow` | 2–4 top-level buckets, flat 3-step plan | Smaller or faster models, tight context, localized bugs |
| `depth=2` | `standard` (default) | 2 levels of branches, full plan with verification gates | General engineering, multi-file debugging, system configuration |
| `depth=3` | `deep` | 3 levels, cross-branch dependency check, nested plan with rollback rules | High-reasoning models, distributed systems, architecture |

```text
# Standard (depth=2)
/spd Diagnose why local inference throughput drops by 60% under concurrent requests

# Shallow (depth=1) for fast or smaller models
/spd depth=1 Fix the systemd unit restart loop on service X

# Deep (depth=3) for large reasoning models
/spd depth=3 Design a zero-downtime migration plan for the document database schema
```

The skill can also trigger automatically on complex debugging, architecture and root-cause analysis tasks.

## Install

The skill is a single folder containing `SKILL.md`. The folder name must be `spd`.

### Option A — `skills` CLI (any supported agent)

Requires Node.js. The [`skills`](https://github.com/vercel-labs/skills) CLI detects your installed agents (Claude Code, Codex, Cursor, OpenCode and others) and places the skill correctly.

```bash
# Interactive
npx skills add Veos2k/spd

# Non-interactive, Claude Code only
npx skills add Veos2k/spd --agent claude-code --yes

# Global (user-level) instead of project-level
npx skills add Veos2k/spd --agent claude-code --global --yes
```

### Option B — Manual install

Save `SKILL.md` as `spd/SKILL.md` inside your harness's skill directory.

| Harness | Scope | Path |
| :--- | :--- | :--- |
| Claude Code | User (all projects) | `~/.claude/skills/spd/SKILL.md` |
| Claude Code | Project | `.claude/skills/spd/SKILL.md` |
| Hermes | User | `~/.hermes/skills/spd/SKILL.md` |
| Cursor | Project | `.cursor/skills/spd/SKILL.md` |

Clone straight into the skill directory (example for Claude Code, user scope):

```bash
git clone https://github.com/Veos2k/spd ~/.claude/skills/spd
```

Or download only the one file:

```bash
mkdir -p ~/.claude/skills/spd
curl -fsSL https://raw.githubusercontent.com/Veos2k/spd/main/SKILL.md \
  -o ~/.claude/skills/spd/SKILL.md
```

Replace `~/.claude/skills` with the path from the table for another harness.

### Option C — Claude.ai / Claude Desktop

Create a ZIP that contains a top-level `spd/` folder with `SKILL.md` inside, then upload it under **Settings → Capabilities → Skills**.

```bash
mkdir -p build/spd && cp SKILL.md build/spd/ && (cd build && zip -r ../spd.zip spd)
```

### Verify

Restart or reload your agent, then run:

```text
/spd depth=1 Explain why a for-loop over a list I am mutating skips items
```

The agent should open its answer with the scaffold `[SPD | Depth: 1 | Cynefin: ...]`.

### Update and uninstall

```bash
# Update (git install)
git -C ~/.claude/skills/spd pull --ff-only

# Update / remove (skills CLI install)
npx skills update spd
npx skills remove spd

# Uninstall (manual install)
rm -rf ~/.claude/skills/spd
```

## For AI agents

If you are an agent asked to install this skill, follow [`AGENTS.md`](AGENTS.md). It contains deterministic steps, a path table and a verification gate.

## Repository layout

```text
spd/
├── SKILL.md     # the skill itself (YAML frontmatter + protocol)
├── README.md    # this file: overview and install instructions
├── AGENTS.md    # install instructions for agents
└── LICENSE      # Apache-2.0
```

## License

[Apache-2.0](LICENSE)
