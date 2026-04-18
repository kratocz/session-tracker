# CLAUDE.md

## Plugin overview

`session-tracker` is a Claude Code plugin for tracking work sessions in Toggl Track or Clockify.

## Structure

```
session-tracker/
├── .claude-plugin/
│   └── plugin.json        ← plugin manifest
├── skills/
│   ├── setup-tracker/
│   │   └── SKILL.md       ← /setup-tracker skill
│   ├── start/
│   │   └── SKILL.md       ← /start skill
│   └── stop/
│       └── SKILL.md       ← /stop skill
├── README.md
└── CLAUDE.md
```

## Config file

Skills read/write `~/.claude/plugins/session-tracker/config.json`:

```json
{
  "backend": "toggl",
  "billable": true,
  "language": "en",
  "toggl": {
    "api_key": "...",
    "workspace_id": 1234567,
    "default_project_id": null
  }
}
```

Top-level fields apply regardless of backend. Defaults when missing: `billable` → `true`, `language` → `"en"`. `language` influences Claude-generated text (prompts, confirmations, descriptions derived from a URL title); it is **not** sent to the tracker API.

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `/setup-tracker` | First run / reconfigure | Interactive setup, writes config |
| `/start [desc]` | Begin tracking | Starts timer via API |
| `/stop` | End tracking | Stops running timer, reports duration |

## Adding a new backend

1. Add a section to `config.json` (e.g. `"harvest": { ... }`)
2. Add `backend: "harvest"` handling to each skill
3. Bump version in `plugin.json`

## Release workflow

After merging to `main`, each version bump follows the same pattern:

1. Bump `version` in `plugin.json` **and** in every skill's frontmatter (keep them in sync).
2. Commit the bump (`feat:` / `fix:` / `chore:` with a body describing changes).
3. Create an annotated tag: `git tag -a vX.Y.Z -m "Release vX.Y.Z: <summary>"`.
4. Push both: `git push origin main && git push origin vX.Y.Z`.
5. Create a GitHub Release from the tag with `gh release create vX.Y.Z --title "vX.Y.Z — <summary>" --notes "$(cat <<'EOF' ... EOF)"`. Release notes should summarize user-visible changes (markdown, bulleted).

SemVer: patch for hardening/metadata, minor for new config fields or behavior, major for breaking changes (config schema changes that break existing configs).
