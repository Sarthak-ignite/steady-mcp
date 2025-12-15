# Troubleshooting (steady-mcp)

This MCP server automates Steady via the web UI. If something changes on Steady, small HTML/CSRF updates may be needed.

---

## 1) “Tool not found” in Cursor

Symptoms:
- Cursor can’t see `steady_login`, `steady_submit_checkin`, etc.

Checklist:
- Fully quit Cursor (Cmd+Q / Exit), then reopen.
- Verify your `mcp.json` entry points to an **absolute path** for `tools/steady-mcp/src/index.js`.
- Verify Node is available:

```bash
node -v
```

- Verify dependencies are installed:

```bash
cd tools/steady-mcp
npm install
```

---

## 2) Server fails to start

Run a quick manual probe (prints tools list) from the repo:

```bash
cd tools/steady-mcp
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | node src/index.js
```

If you see JSON with a `tools` array, the server starts correctly.

---

## 3) Login fails (`steady_login`)

Common causes:
- Wrong `STEADY_EMAIL` / password
- Your Steady account uses a different auth flow (SSO / magic link)

Fixes:
- If your Steady flow is not email→password, use `steady_set_cookies` instead (see README).
- If your org uses email→password but the tool fails, enable debug:
  - set `STEADY_MCP_DEBUG=1` in the MCP env and retry

Security warning:
- Debug output may include cookies. Don’t paste logs into public channels.

---

## 4) `steady_submit_checkin` returns HTTP 422

422 means Steady rejected the update.

Most common reasons:
- The team is already checked in for that date
- Session/CSRF mismatch (usually fixed by cookie jar, but re-login helps)
- You don’t have permission to check in to that team
- Required fields changed on Steady’s side

Recommended recovery steps:
1) Run `steady_login` again.
2) Run `steady_list_teams` and confirm the team name is correct.
3) Try again with the same payload.
4) If still 422, open Steady in browser and verify the team is actually pending for today.

---

## 5) “Signed out” / redirects to signin

Symptoms:
- `steady_ping` returns `{ ok: false }`

Fix:
- run `steady_login` again
- if login automation doesn’t work for your account, use `steady_set_cookies`

---

## 6) Where are cookies stored?

See README for OS-specific defaults.

You can override paths:
- `STEADY_COOKIES_PATH`
- `STEADY_COOKIE_JAR_PATH`


