# full-send

An agent skill that finishes a coding task. The agent fixes it, verifies the fix, opens a pull request, stays until CI and the auto-reviewers are clean, checks the diff one last time, and merges only when its confidence is high.

This is an independent write-up of the workflow Theo (t3.gg) described on [Nerd Snipe](https://www.youtube.com/shorts/OdYpxJ9T6TI) in October 2026. He has not published his own skill file. The words in `SKILL.md` are not his.

The person who invokes it is assumed to never look at the pull request. The agent leaves the pull request open when the task is ambiguous, verification cannot be run, branch protection blocks the merge, or confidence is not high.

## Install

Put this folder where your agent reads skills, under the name `full-send`. A symlink keeps `git pull` as the update.

```sh
git clone https://github.com/Samin12/full-send.git ~/Projects/full-send
ln -s ~/Projects/full-send ~/.claude/skills/full-send
ln -s ~/Projects/full-send ~/.agents/skills/full-send
mkdir -p ~/.grok/skills
ln -s ~/Projects/full-send ~/.grok/skills/full-send
```

| Agent | Start it |
|---|---|
| Claude Code, Cursor, Grok | `/full-send` and the task |
| Codex | `$full-send` and the task |
| Any agent that reads `SKILL.md` | say "full send" and the task |

The skill does not auto-invoke. Merging is opt-in: name `full send` or `/full-send`.

The agent needs `git`, the GitHub CLI (`gh auth login`), and permission to push and merge in that repo.

## Use

```
/full-send <the task>
```

When it finishes, the report is the pull request URL, the merge commit, what it verified, and the residual risk. When it stops short, the report is the open pull request URL and the specific doubt.
