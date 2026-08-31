# ny-skills

Shared opencode workflow assets for the ny pipeline. Repo layout mirrors
`~/.config/opencode/` one-to-one, so every path here has an obvious install
destination.

## Layout

```
agent/                  → ~/.config/opencode/agent/
  pr-reviewer.md          Read-only PR-review subagent (deepseek-v4-pro).
                          Reviews a GitHub PR using the built-in /review
                          methodology, posts one findings comment, returns a
                          JSON verdict. Used by ny-pipeline step 7.
ny-skills/              → ~/.config/opencode/ny-skills/
  ny-pipeline.skill.md    Orchestrator: Plan → Implement → Verify → internal
                          review → PR → post-PR external review loop (step 7,
                          spawns pr-reviewer, max 2 rounds).
  ny-fetch / ny-plan / ny-implement / ny-verify / ny-audit /
  ny-coverage / ny-review-docs / ny-document / ny-create-pr
                          Individually callable steps followed inline by
                          pipeline subagents.
  conventional-commit.skill.md
                          Commit-message convention used by pipeline commits.
  issue.skill.md          Open a GitHub issue through the pipeline as a skill
                          (Codex and OpenCode).
command/                → ~/.config/opencode/command/
  issue.md                OpenCode `/issue <number>` slash command: kick off
                          the pipeline for a GitHub issue.
skills/                 → ~/.codex/skills/ (Codex-native layout)
  <name>/SKILL.md         Same 12 skills as `ny-skills/`, laid out as
                          Codex skill directories. Linked into
                          `~/.codex/skills/<name>` per worktree so Codex
                          discovers them (Codex skips file symlinks, so the
                          directory itself is the symlink).
```

Not tracked here: `opencode.jsonc`, `plugins/` — machine-local
config that may contain secrets. Keep those out of git.

## Install / sync

Copy each directory onto the matching path under `~/.config/opencode/`:

```bash
git clone git@github.com:nomanyaqub/ny-skills.git
cp -r ny-skills/agent    ~/.config/opencode/
cp -r ny-skills/ny-skills ~/.config/opencode/
```

The local machine is the source of truth while iterating; push changes back
here after they stabilise.
