---
name: start
description: Start a time tracking session or timer. Use when the user says "/start", "start session", "start timer", "begin tracking", "I'm starting work on...", or any similar phrase indicating they want to begin tracking time.
argument-hint: [task-description-or-url]
version: 1.2.0
allowed-tools: Read, Bash
---

# Start Session

Start a new time tracking session in the configured backend.

## Steps

1. **Read config**: Read `~/.claude/plugins/session-tracker/config.json` using the Read tool.
   - If the file doesn't exist: "No configuration found. Please run /setup-tracker first." Then stop.

2. **Gather project context** (best-effort — ignore errors if not in a git repo):
   ```bash
   git remote get-url origin 2>/dev/null
   git branch --show-current 2>/dev/null
   pwd
   ```
   Derive a project name: take the last path segment of the git remote URL and strip a trailing `.git` (e.g., `git@github.com:user/my-repo.git` → `my-repo`). If not a git repo, fall back to the current directory name.

3. **Check for an already-running timer**:

   ### Toggl Track
   ```bash
   curl -s -u "<config.toggl.api_key>:api_token" \
     https://api.track.toggl.com/api/v9/time_entries/current
   ```

   ### Clockify
   ```bash
   curl -s -H "X-Api-Key: <config.clockify.api_key>" \
     "https://api.clockify.me/api/v1/workspaces/<workspace_id>/time-entries?in-progress=true&page-size=1"
   ```

   If a timer is already running, show its description and ask the user whether to stop it first (via `/stop`) or keep it running. Do not auto-stop.

4. **Determine description**:
   - If arguments were provided, use them as-is. Accepts plain text or a URL — pass through unchanged.
   - Otherwise, ask the user for a brief description of what they're working on.

5. **Resolve tracker project** (optional):
   - Fetch the active project list and look for a project whose name matches the detected repo/dir name (case-insensitive).
   - If matched, use its ID. Otherwise fall back to `default_project_id` from config (may be null — in which case no project is attached).

   ### Toggl Track
   ```bash
   curl -s -u "<config.toggl.api_key>:api_token" \
     "https://api.track.toggl.com/api/v9/workspaces/<workspace_id>/projects?active=true"
   ```

   ### Clockify
   ```bash
   curl -s -H "X-Api-Key: <config.clockify.api_key>" \
     "https://api.clockify.me/api/v1/workspaces/<workspace_id>/projects"
   ```

6. **Get current UTC time**:
   ```bash
   date -u +%Y-%m-%dT%H:%M:%SZ
   ```

7. **Start timer** based on `config.backend`. Include the project ID field only if resolved in step 5; otherwise omit it entirely.

   ### Toggl Track
   ```bash
   curl -s -u "<config.toggl.api_key>:api_token" \
     -H "Content-Type: application/json" \
     -X POST "https://api.track.toggl.com/api/v9/time_entries" \
     -d '{"description":"<description>","workspace_id":<workspace_id>,"start":"<UTC time>","duration":-1,"created_with":"session-tracker-claude-plugin"}'
   ```

   ### Clockify
   ```bash
   curl -s -H "X-Api-Key: <config.clockify.api_key>" \
     -H "Content-Type: application/json" \
     -X POST "https://api.clockify.me/api/v1/workspaces/<workspace_id>/time-entries" \
     -d '{"description":"<description>","start":"<UTC time>"}'
   ```

8. **Report result** (keep it concise):
   - Success: `Session started: '<description>'` — append ` — project: <name>` if a project was resolved.
   - On HTTP error: show the API error message and suggest re-running `/setup-tracker`.
