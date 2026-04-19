# skill-pack

A curated set of [Claude Code](https://claude.com/claude-code) skills, installable as a single plugin.

```bash
/plugin marketplace add valetivivek/skill-pack
/plugin install skill-pack@skill-pack
```

Starts small and grows. Every skill here is one I actually use. If a skill stops earning its slot, it gets cut.

## Skills

| Skill | Triggers on | What you get |
|-------|-------------|--------------|
| [`handoff`](skills/handoff/SKILL.md) | "handoff", "wrap up", "end of session", "pause here", "I'm done for today" | A timestamped markdown file at `./handoff/YYYY-MM-DD-HHMM-slug.md` capturing what was done, what's left, known issues, key decisions, and a copy-pasteable resume prompt for the next session (same agent or different). |

More skills land as they get written. See [Roadmap](#roadmap) below.

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI, desktop, web, or IDE extension)
- A project directory. Most skills work best inside a git repo, but none of them require one.

Tested against Claude Code as of April 2026. Skills are plain markdown, so forward compatibility should be fine.

## How it works

A Claude Code skill is a markdown file with YAML frontmatter. The frontmatter tells the model when to trigger (the `description` field) and the body tells it what to do. Installing this plugin drops the skills into Claude Code's skill registry. No binaries, no runtime, no background process. The model reads the `SKILL.md` when the trigger phrase shows up in your conversation, and follows its instructions.

## Install

```bash
/plugin marketplace add valetivivek/skill-pack
/plugin install skill-pack@skill-pack
```

Verify the install by asking Claude Code to list available skills, or by typing one of the trigger phrases (e.g. "handoff this session") and watching it fire.

## Update

```bash
/plugin marketplace update skill-pack
```

## Uninstall

```bash
/plugin uninstall skill-pack@skill-pack
/plugin marketplace remove skill-pack
```

## Testing a skill locally

If you're developing a skill in this repo (or want to try changes before pushing), install the plugin from your local clone instead of GitHub:

```bash
/plugin marketplace add /absolute/path/to/skill-pack
/plugin install skill-pack@skill-pack
```

Edit the `SKILL.md`, reload Claude Code, retrigger. No rebuild step.

## Repository layout

```
skill-pack/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest (name, version, skills glob)
│   └── marketplace.json    # marketplace manifest (so users can `marketplace add` this repo)
├── skills/
│   └── handoff/
│       └── SKILL.md        # one folder per skill, one SKILL.md inside
├── .gitignore
├── LICENSE
└── README.md
```

One folder per skill under `skills/`. The plugin manifest globs the whole `skills/` directory, so new skills are picked up by adding a folder and committing.

## Roadmap

Skills currently on the shortlist (no promises on order):

- `pr-review-prep` — scan a branch and draft a self-review before opening the PR
- `daily-log` — append a dated entry to a project journal with what changed today
- `unstuck` — structured "I'm stuck" prompt to reset a session without losing context
- `repo-snapshot` — one-page summary of a repo for pasting into an external chat

Open an issue if you want to add to this list or vote one up.

## Versioning

`skill-pack` follows semver. Skill additions are minor bumps, skill removals or breaking trigger changes are major bumps, fixes are patches. The full version history lives in git tags.

## Credits

Inspired by [everything-claude-code](https://github.com/affaan-m/everything-claude-code) and [superpowers](https://github.com/obra/superpowers), which set a high bar for what a Claude Code skill pack can be.

## License

MIT. See [LICENSE](LICENSE).
