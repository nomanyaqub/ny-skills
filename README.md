# ny-skills

Shared workflow assets for the ny pipeline, split per coding agent via git
worktrees. The `opencode` branch (this worktree) holds only what opencode
consumes; the `codex` branch holds Codex-native skills.

## Branch ↔ worktree ↔ config

| Branch     | Worktree                              | Installed into        |
| ---------- | ------------------------------------- | --------------------- |
| `opencode` | `~/workspace/github/ny-skills-opencode` | `~/.config/opencode/` |
| `codex`    | `~/workspace/github/ny-skills-codex`    | `~/.codex/skills/`    |
| `main`     | `~/workspace/github/ny-skills`          | shared base           |

This worktree contains only the opencode structure: `agent/`, `command/`,
`ny-skills/`. The Codex-native `skills/` directory layout lives on the
`codex` branch, not here.

## Layout

```
agent/                  → ~/.config/opencode/agent/
  ny-implement.md         Pipeline phase implementer (opencode-go/deepseek-v4-flash).
  ny-plan.md              Pipeline planner (opencode-go/glm-5.3-flash).
  ny-reviewer.md          Internal review subagent (opencode-go/glm-5.3-flash).
  pr-reviewer.md          Read-only PR-review subagent (opencode-go/glm-5.3-flash).
                          Reviews a GitHub PR using the built-in /review
                          methodology, posts one findings comment, returns a
                          JSON verdict. Used by ny-pipeline step 7.
ny-skills/              flat skill sources; each exposed individually as
                          ~/.config/opencode/skills/<name>/SKILL.md (symlink)
  ny-pipeline.skill.md    Orchestrator: Plan → Implement → Verify → internal
                          review → PR → post-PR external review loop (step 7,
                          spawns pr-reviewer, max 2 rounds).
  ny-fetch / ny-plan / ny-implement / ny-verify / ny-audit /
  ny-coverage / ny-review-docs / ny-document / ny-create-pr
                          Individually callable steps followed inline by
                          pipeline subagents.
  conventional-commit.skill.md
                          Commit-message convention used by pipeline commits.
  issue.skill.md          Open a GitHub issue through the pipeline as a skill.
command/                → ~/.config/opencode/command/
  issue.md                `/issue <number>` slash command: kick off the
                          pipeline for a GitHub issue.
```

Not tracked here: `opencode.json`, `opencode.jsonc`, `plugins/` —
machine-local config that may contain secrets. Keep those out of git.

## Install / sync

Everything is symlinked, not copied, so edits here take effect on the next
session start:

```bash
ln -sfn ~/workspace/github/ny-skills-opencode/agent     ~/.config/opencode/agent
ln -sfn ~/workspace/github/ny-skills-opencode/ny-skills ~/.config/opencode/ny-skills
for f in ~/workspace/github/ny-skills-opencode/ny-skills/*.skill.md; do
  name=$(basename "$f" .skill.md)
  mkdir -p ~/.config/opencode/skills/$name
  ln -sf "$f" ~/.config/opencode/skills/$name/SKILL.md
done
mkdir -p ~/.config/opencode/command
ln -sf ~/workspace/github/ny-skills-opencode/command/issue.md ~/.config/opencode/command/issue.md
```

The local machine is the source of truth while iterating; push changes back
here after they stabilise.

## Skill frontmatter rules

Skill frontmatter is parsed as YAML. An unquoted `: ` inside a
`description:` value is invalid YAML and opencode silently drops the skill
at session start; a ` #` in the value truncates it (YAML comment). Quote
descriptions that contain either:

```yaml
description: "Run the pipeline for an issue: fetch, plan, implement, verify."
```
