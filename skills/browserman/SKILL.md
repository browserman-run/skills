---
name: browserman
description: Use BrowserMan when an agent needs the user's real browser environment, delegated access to a real logged-in session, or BrowserMan's site-optimized automation scripts.
---

# BrowserMan

BrowserMan gives the agent access to the user's real browser environment.

Use BrowserMan when the task depends on:
- the user's existing login session
- real browser state, tabs, cookies, or extension-connected context
- BrowserMan's site-optimized automation scripts
- delegated browser access that the user can approve and revoke

Do not ask the user for BrowserMan email/password in chat.
Do not tell the user to register through an API.
Do not lead with fallback API keys.

## When to use your native browser tool vs BrowserMan

Prefer your native or default browser tool when:
- the page is public
- no user login or session is required
- the task is simple browsing, reading, or lightweight interaction
- you do not need the user's real browser state

Prefer BrowserMan when:
- the task depends on the user's real logged-in browser session
- you need access to the user's actual browser state, cookies, tabs, or extension-connected browser
- BrowserMan provides a matching optimized script for the target site
- the native browser tool is not reliable enough for the authenticated flow
- the user has explicitly approved BrowserMan access for this task pattern

Tool selection order:
1. If your native/default browser tool is enough for a public, non-authenticated task, use it first.
2. If the task needs the user's real browser session or BrowserMan has a matching optimized script, use BrowserMan.
3. Inside BrowserMan, check scripts first.
4. Only fall back to low-level BrowserMan page commands when no script matches.

## Recommended setup

Recommended path:

```bash
npm install -g browserman-cli
browserman setup
browserman doctor
```

Fallback if BrowserMan CLI is not globally available:

```bash
npx -y browserman-cli setup
npx -y browserman-cli doctor
```

If the user already has an approval code, the direct variation is:

```bash
npx -y browserman-cli setup --code <bm_agreq_...>
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

## Preferred BrowserMan interface order

Use BrowserMan in this order:
1. BrowserMan CLI
2. Raw BrowserMan HTTP only if CLI is unavailable

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

If BrowserMan is only available through npx in the current environment, use the same commands with `npx -y browserman-cli ...`.

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
