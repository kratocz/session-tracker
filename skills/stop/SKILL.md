---
name: stop
description: Stop the current time tracking session or timer. Takes no arguments — always stops the currently running entry. Use when the user says "/stop", "stop session", "stop timer", "end session", "I'm done", "finished working", or any similar phrase indicating they want to stop tracking time.
version: 1.2.0
allowed-tools: Read, Bash
---

# Stop Session

Stop the currently running time tracking session and report its duration.

## Steps

1. **Read config**: Read `~/.claude/plugins/session-tracker/config.json` using the Read tool.
   - If the file doesn't exist: "No configuration found. Please run /setup-tracker first." Then stop.

2. **Get the current running timer**:

   ### Toggl Track
   ```bash
   curl -s -u "<config.toggl.api_key>:api_token" \
     https://api.track.toggl.com/api/v9/time_entries/current
   ```
   If the response body is `null`: "No session is currently running." Then stop.

   ### Clockify
   ```bash
   curl -s -H "X-Api-Key: <config.clockify.api_key>" \
     "https://api.clockify.me/api/v1/workspaces/<workspace_id>/time-entries?in-progress=true&page-size=1"
   ```
   If the array is empty: "No session is currently running." Then stop.

3. **Stop the timer**:

   ### Toggl Track
   ```bash
   curl -s -u "<config.toggl.api_key>:api_token" \
     -H "Content-Type: application/json" \
     -X PATCH "https://api.track.toggl.com/api/v9/time_entries/<workspace_id>/<timer_id>/stop"
   ```

   ### Clockify
   Get current UTC time first:
   ```bash
   date -u +%Y-%m-%dT%H:%M:%SZ
   ```
   Then stop:
   ```bash
   curl -s -H "X-Api-Key: <config.clockify.api_key>" \
     -H "Content-Type: application/json" \
     -X PATCH "https://api.clockify.me/api/v1/workspaces/<workspace_id>/user/<user_id>/time-entries" \
     -d '{"end":"<current UTC time>"}'
   ```

4. **Calculate and report duration**:
   - Parse the `start` timestamp from the timer response.
   - Calculate elapsed time (hours and minutes).
   - Report: `Session stopped: <Xh Ym> — '<description>'`
   - Example: `Session stopped: 1h 23m — 'Refactoring auth module'`
