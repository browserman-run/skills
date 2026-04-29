# BrowserMan Skills

Open BrowserMan skills for the open agent skills ecosystem.

Install with:

```bash
npx skills add browserman-run/skills
```

This repository contains reusable skills that teach coding agents how to use BrowserMan through its delegated setup flow, CLI, and MCP bridge. The npm package is `browserman-cli`; after global install, the command agents should run is `browserman`. For one-off usage, use `npx -y browserman-cli <subcommand>`.

Current contents:
- `browserman` — Browser automation through BrowserMan with delegated auth, CLI-first setup, and MCP support

Recommended next step after installing the skill:

```bash
npx -y browserman-cli setup
```

Then validate with:

```bash
npx -y browserman-cli doctor
```
