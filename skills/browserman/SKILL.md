---
name: browserman
description: Use BrowserMan when you want a coding agent to automate a real browser with delegated auth, BrowserMan CLI, or MCP. Prefer built-in BrowserMan scripts first, then fall back to low-level browser commands.
---

# BrowserMan

Use BrowserMan to control a real browser through delegated auth.

## When to use this skill

Use this skill when:
- you need a coding agent to work in a real browser
- you want to reuse a signed-in browser session
- you want browser access scoped through BrowserMan approval
- you want to drive BrowserMan through CLI first, with MCP as an optional host integration

Do not ask the user for BrowserMan email/password in chat.
Do not ask the user to register through an API endpoint.
Do not lead with fallback API keys.

## Standard setup path

The standard path is:

1. Install the BrowserMan CLI if needed
2. Run:

```bash
npx browserman-cli setup
```

3. Approve the request in BrowserMan on the web
4. Validate the saved delegated setup:

```bash
npx browserman-cli doctor
```

BrowserMan stores delegated local config at:

```text
~/.browserman/config.json
```

Treat that file as the local source of truth for:
- serverUrl
- token
- tokenType
- browserId
- browserIds
- browserScopeMode
- capabilities

## Preferred interface order

Use BrowserMan in this order:

1. BrowserMan CLI
2. BrowserMan MCP only when your host expects MCP
3. Raw BrowserMan HTTP only if CLI is unavailable

Prefer commands like:

```bash
browserman browser list --json
browserman browser current --json
browserman browser ping --json
browserman page open --url https://example.com --json
browserman page read --json
browserman page click --ref 12 --json
browserman page type --text "hello" --json
browserman page press --key Enter --json
browserman page screenshot --out ./page.png --json
browserman script list --json
browserman script run --site x.com --action search --text "browserman" --json
```

## MCP usage

If your host expects MCP, first complete delegated setup, then configure BrowserMan MCP with:

```json
{
  "mcpServers": {
    "browserman": {
      "command": "browserman",
      "args": ["mcp"]
    }
  }
}
```

If BrowserMan is not installed globally, use a one-off command form that your MCP host supports, for example:

```json
{
  "mcpServers": {
    "browserman": {
      "command": "npx",
      "args": ["-y", "browserman-cli", "mcp"]
    }
  }
}
```

## Recovery commands

If setup is missing or broken, suggest exactly:

```bash
npx browserman-cli setup
npx browserman-cli doctor
```

If the user already has an approval code, the direct variation is:

```bash
npx browserman-cli setup --code <bm_agreq_...>
```

Expected token families:
- approval request code: `bm_agreq_...`
- delegated runtime token: `bm_dlg_...`

## Capability-scoped tokens

Delegated tokens can be scoped by capability.

Important capabilities:
- `observe` — browser listing, status, ping, execution reads, page inspection
- `commands` — navigate, click, type, press, run_script, low-level browser control
- `browserManagement` — browser creation, update, delete, and other management operations

When a BrowserMan call returns 403:
- do not assume the token is invalid
- first suspect missing capability or browser out of scope
- ask the user to re-approve in BrowserMan if needed

## Required execution order

Always follow this order.

### Step 0: Check the BrowserMan script catalog first

Before low-level browser control, list scripts:

```bash
browserman script list --json
```

If a matching platform/action exists, prefer `browserman script run`.

### Step 1: Confirm browser availability with ping

Before real work, verify the chosen browser is reachable:

```bash
browserman browser ping --json
```

If this returns offline or 503, the browser is not currently connected.

### Step 2A: If a script matches, run it

Example:

```bash
browserman script run --site x.com --action search --text "browserman" --json
```

### Step 2B: If no script matches, use low-level page commands

Recommended low-level sequence:
1. `browserman page open --url ... --json`
2. `browserman page read --json`
3. `browserman page click` / `browserman page type` / `browserman page press`
4. `browserman page read --json` again after page changes
5. `browserman page screenshot --out ./page.png --json` or `browserman page url --json` to verify state

## Destructive-action checklist

Before performing destructive or externally visible actions, confirm intent with the user.

Examples:
- delete
- purchase
- submit
- post
- publish
- send message
- irreversible settings changes

Especially confirm before:
- clicking destructive UI
- running BrowserMan scripts that post, delete, or change account state
- low-level form submission that causes side effects

## What not to do

Do not:
- ask the user for BrowserMan email/password in chat
- tell the user to register through an API endpoint
- tell the user to create a fallback key as the default onboarding path
- assume every browser is in scope for the delegated token
- assume every delegated token has `commands`
- skip the script-catalog check
