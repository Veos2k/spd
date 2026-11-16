# AGENTS.md — installing the `spd` skill

This file is for AI agents. Humans should read [`README.md`](README.md).

## What this repository is

- One skill: **SPD (Systematic Problem Decomposer)**.
- Entry point: `SKILL.md` at the repository root (YAML frontmatter `name: spd`, then the protocol).
- Invocation: `/spd [depth=1|2|3] <problem or task>`. Default depth is 2.
- Source: `https://github.com/Veos2k/spd` (branch `main`).

## Install procedure

Goal: the file `SKILL.md` exists at `<SKILLS_DIR>/spd/SKILL.md`.

1. **Choose `SKILLS_DIR`.** Use the first row that matches the harness you are running in. If the user named a scope (user-level or project-level), follow it.

   | Harness | User scope | Project scope |
   | :--- | :--- | :--- |
   | Claude Code | `~/.claude/skills` | `.claude/skills` |
   | Hermes | `~/.hermes/skills` | not applicable |
   | Cursor | not applicable | `.cursor/skills` |

   If the harness is not listed, look in your own documentation for the skill directory and use that. Do not guess.

2. **Install.** Prefer the `skills` CLI when Node.js is available; otherwise use git; otherwise use curl.

   ```bash
   # CLI (Claude Code shown; replace --agent as needed, add --global for user scope)
   npx skills add Veos2k/spd --agent claude-code --yes

   # git
   DEST="$SKILLS_DIR/spd"
   mkdir -p "$SKILLS_DIR"
   if [ -d "$DEST/.git" ]; then
     git -C "$DEST" pull --ff-only
   else
     git clone --depth 1 https://github.com/Veos2k/spd "$DEST"
   fi

   # curl (single file)
   mkdir -p "$SKILLS_DIR/spd"
   curl -fsSL https://raw.githubusercontent.com/Veos2k/spd/main/SKILL.md \
     -o "$SKILLS_DIR/spd/SKILL.md"
   ```

3. **Verification gate.** All checks must pass before you report success.

   ```bash
   test -f "$SKILLS_DIR/spd/SKILL.md" \
     && grep -q '^name: spd$' "$SKILLS_DIR/spd/SKILL.md" \
     && echo OK
   ```

   Expected output: `OK`.

4. **Activate.** Most harnesses load skills at session start. Tell the user to restart or reload the agent if `/spd` is not recognised yet.

## Rules

- The folder name must be exactly `spd`. It must match `name:` in the frontmatter.
- Do not edit `SKILL.md` while installing. Install it as published.
- Do not install to a scope the user did not ask for when they gave one.
- If a step fails twice, stop and report the exact error. Do not try unrelated workarounds.
- Uninstall: delete the `spd/` folder from `SKILLS_DIR` (or run `npx skills remove spd`).

## Using the skill after install

When the user invokes `/spd` or the task is complex debugging, architecture or root-cause analysis:

1. Read `SKILL.md` and follow its phases in order.
2. Parse the depth argument (`depth=1|2|3`, `shallow`, `deep`). Default is `depth=2`.
3. Begin the answer with the compact scaffold from section 4 of `SKILL.md`.
