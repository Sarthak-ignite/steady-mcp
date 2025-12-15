# Steady MCP (local)

This is a **local MCP server** that submits Steady check-ins by driving Steady’s **web form** (there is **no official Steady API**).

It’s built for the workflow:

> You tell the AI: **team + check-in text** → AI calls the MCP tool → Steady check-in is submitted.

---

## What this supports

- **Email + password login** (two-step flow from [`/signin`](https://app.steady.space/signin)) via `steady_login`
- **Cookie-based auth** via `steady_set_cookies` (fallback for non-standard auth flows)
- **Team discovery** via `steady_list_teams`
- **Submitting a check-in for one team** via `steady_submit_checkin`

### How it works (important)

Steady rotates the `_sthr_session` cookie when you load the daily edit page (`/check-ins/YYYY-MM-DD/edit`).  
So **this server uses a curl cookie jar** (`cookiejar.txt`) to preserve the updated session cookie between the GET and POST.

---

## Requirements

- **Node.js 18+**
- **Cursor** (or another MCP client) with support for running local MCP servers over stdio
- Access to Steady web app (`https://app.steady.space`)

---

## Install

```bash
cd tools/steady-mcp
npm install
```

---

## Configure Cursor (MCP)

Add a server entry to your Cursor MCP config.

### Where is `mcp.json`?

Common locations:
- **macOS / Linux**: `~/.cursor/mcp.json`
- **Windows**: `%USERPROFILE%\\.cursor\\mcp.json`

If you don’t see it, use Cursor’s UI settings for MCP servers (recommended), or search for `mcp.json` on your machine.

### Quickstart: macOS (recommended)

1) **Install Node.js 18+**

- If you use Homebrew:

```bash
brew install node
node -v
```

2) **Install dependencies**

```bash
cd <ABS_PATH_TO_REPO>/tools/steady-mcp
npm install
```

3) **Add `steady-mcp` to `~/.cursor/mcp.json`** (example below)

4) **Restart Cursor** (quit + reopen)

5) In Cursor, run:
- `steady_login`
- `steady_submit_checkin`

### Quickstart: Windows

1) **Install Node.js 18+**

- If you use `winget`:

```powershell
winget install OpenJS.NodeJS.LTS
node -v
```

2) **Install dependencies**

```powershell
cd <ABS_PATH_TO_REPO>\tools\steady-mcp
npm install
```

3) **Add `steady-mcp` to `%USERPROFILE%\.cursor\mcp.json`**

4) **Restart Cursor** (fully quit + reopen)

5) In Cursor, run:
- `steady_login`
- `steady_submit_checkin`

### Example `mcp.json` snippet

> Replace `<ABS_PATH_TO_REPO>` and your email.

```json
{
  "mcpServers": {
    "steady-mcp": {
      "command": "node",
      "args": [
        "<ABS_PATH_TO_REPO>/tools/steady-mcp/src/index.js"
      ],
      "env": {
        "STEADY_BASE_URL": "https://app.steady.space",
        "STEADY_EMAIL": "you@company.com",
        "STEADY_PASSWORD": "<YOUR_PASSWORD>",
        "STEADY_MCP_DEBUG": "0"
      }
    }
  }
}
```

### Windows path tip (important)

In JSON, Windows backslashes require escaping. Easiest option: use forward slashes in the `args` path:

```json
{
  "mcpServers": {
    "steady-mcp": {
      "command": "node",
      "args": [
        "C:/Users/<YOU>/path/to/repo/tools/steady-mcp/src/index.js"
      ],
      "env": {
        "STEADY_BASE_URL": "https://app.steady.space",
        "STEADY_EMAIL": "you@company.com",
        "STEADY_PASSWORD": "<YOUR_PASSWORD>"
      }
    }
  }
}
```

### Security note

- `STEADY_PASSWORD` in `mcp.json` is the simplest setup, but it’s **not** ideal for security.
- Prefer `STEADY_PASSWORD_COMMAND` (password retrieved at runtime) whenever possible.

---

## Credentials / Secrets (cross-platform)

The server reads credentials from environment variables provided by your MCP client.

### Option A (simple): plaintext env password

- `STEADY_EMAIL`
- `STEADY_PASSWORD`

### Option B (recommended): password command

- `STEADY_EMAIL`
- `STEADY_PASSWORD_COMMAND` — command that prints the password to stdout (no prompts)

Examples:

- **macOS Keychain** (recommended on macOS)
  1) Store once:

```bash
security add-generic-password -a "you@company.com" -s "steady-mcp" -w "<YOUR_PASSWORD>" -U
```

  2) In MCP env:
     - `STEADY_PASSWORD_COMMAND=security find-generic-password -w -s steady-mcp -a you@company.com`

- **Windows**: use a password manager CLI (e.g. 1Password CLI / Bitwarden CLI) or a small local script that prints the password.
- **Any OS**: use your password manager’s CLI, or a small script you keep outside git.

### Option C (macOS-only fallback): Keychain lookup without a command

If you have a Keychain entry, the server can read it automatically on macOS:

- `STEADY_EMAIL=you@company.com`
- optionally `STEADY_KEYCHAIN_SERVICE=steady-mcp`
- optionally `STEADY_KEYCHAIN_ACCOUNT=you@company.com`

---

## Authentication options

### 1) Login automation (email → password)

Use:
- `steady_login`

This will log in and write:
- `cookies.txt` (Cookie header string)
- `cookiejar.txt` (curl jar; used internally to handle session rotation)

### 2) Cookie-based auth (fallback)

If login automation doesn’t work for your Steady account, you can set cookies from your browser:

1) In Steady (browser): DevTools → Application/Storage → Cookies → `https://app.steady.space`
2) Copy these cookies:
   - `_sthr_session`
   - `remember_user_token` (if present)
3) Build a Cookie header string:

```text
_sthr_session=...; remember_user_token=...
```

4) Call:
- `steady_set_cookies` with `{ "cookies": "_sthr_session=...; remember_user_token=..." }`

---

## File locations (where cookies are stored)

Defaults (can be overridden via env):

- **macOS**
  - cookies: `~/Library/Application Support/steady-mcp/cookies.txt`
  - jar: `~/Library/Application Support/steady-mcp/cookiejar.txt`
- **Windows**
  - cookies: `%APPDATA%\\steady-mcp\\cookies.txt`
  - jar: `%APPDATA%\\steady-mcp\\cookiejar.txt`
- **Linux**
  - cookies: `$XDG_CONFIG_HOME/steady-mcp/cookies.txt` (or `~/.config/steady-mcp/cookies.txt`)
  - jar: `$XDG_CONFIG_HOME/steady-mcp/cookiejar.txt` (or `~/.config/steady-mcp/cookiejar.txt`)

Overrides:
- `STEADY_COOKIES_PATH`
- `STEADY_COOKIE_JAR_PATH`

---

## Usage (day-to-day)

### Step 0: Restart Cursor

After adding/updating MCP config, fully restart Cursor (quit + reopen).

### Step 1: Login (refresh cookies)

Call:
- `steady_login`

Then verify:
- `steady_ping` → should return `{ "ok": true }`

### Step 2: List team names (optional)

Call:
- `steady_list_teams`

### Step 3: Submit today’s check-in for one team

Call:
- `steady_submit_checkin`

Example:

```json
{
  "team": "Everest AI",
  "text": "No work today",
  "mood": "calm"
}
```

Success is typically:
- `status_code: 302`

---

## Tools (API surface)

- **`steady_login`**: log in and save cookies locally
- **`steady_set_cookies`**: save browser cookies manually
- **`steady_ping`**: validate auth
- **`steady_list_teams`**: list team names/ids from the daily edit page
- **`steady_submit_checkin`**: submit for one team

---

## Troubleshooting

Start here: `docs/TROUBLESHOOTING.md`

Common issues:

- **Tool not found in Cursor**
  - fully restart Cursor
  - verify the `mcp.json` entry path is correct and absolute
  - verify `npm install` was run in `tools/steady-mcp`

- **HTTP 422**
  - usually means the submission was rejected (already checked in / stale session / insufficient permission)
  - run `steady_login` again and retry
  - confirm the team is actually pending on Steady’s daily page

- **Need debug**
  - set `STEADY_MCP_DEBUG=1` (warning: debug output may include sensitive cookies)
