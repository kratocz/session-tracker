---
name: start
description: Start a time tracking session or timer. Use when the user says "/start", "start session", "start timer", "begin tracking", "I'm starting work on...", or any similar phrase indicating they want to begin tracking time.
version: 1.1.1
allowed-tools: Read, Bash
---

# Start Session

Start a new time tracking session in the configured backend.

## Steps

1. **Read config**: Read `~/.claude/plugins/session-tracker/config.json` using the Read tool.
   - If the file doesn't exist: "No configuration found. Please run /setup-tracker first." Then stop.

2. **Get description**: Use the description provided by the user. If none was given, ask for a brief description of what they're working on.

3. **Get current UTC time**:
   ```bash
   date -u +%Y-%m-%dT%H:%M:%SZ
   ```

4. **Start timer** based on `config.backend`:

   ### Toggl Track
   Build the JSON payload. If `default_project_id` is not null, include `"project_id"` in the payload; otherwise omit it.

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

5. **Report result**:
   - Success: `Session started: '<description>' (ID: <id from response>)`
   - On HTTP error: show the API error message and suggest re-running `/setup-tracker`.
