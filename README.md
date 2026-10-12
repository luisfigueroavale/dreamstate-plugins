# Dreamstate plugins

The Dreamstate plugin for Claude Code, Codex and Grok Build. It gives your coding agent Dreamstate's skills and connects it to the hosted Dreamstate MCP server: build workbooks of companies and people, run LinkedIn and email sequences, answer replies, post to LinkedIn and X, and manage your CRM.

You sign in through your browser the first time the agent uses a Dreamstate tool. You never paste an API key.

## Install

The easiest way installs into every supported agent on your machine:

```sh
npx @trydreamstate/cli install
```

Or install into one agent:

```sh
# Claude Code
claude plugin marketplace add luisfigueroavale/dreamstate-plugins
claude plugin install dreamstate@dreamstate

# Codex
codex plugin marketplace add luisfigueroavale/dreamstate-plugins
codex plugin add dreamstate@dreamstate

# Grok Build
grok plugin marketplace add luisfigueroavale/dreamstate-plugins
grok plugin install dreamstate@dreamstate-plugins
```

## What is inside

- `plugins/dreamstate/.mcp.json`: the hosted MCP server at `https://mcp.trydreamstate.com` (OAuth).
- `plugins/dreamstate/skills/`: Dreamstate's skills, for example `dreamstate:list-building`, `dreamstate:outreach` and `dreamstate:first-win`.
- Marketplace files for each agent: `.claude-plugin/`, `.agents/plugins/` and `.grok-plugin/`.

This repository is generated from the Dreamstate CLI build. Do not edit it by hand.

More at https://trydreamstate.com.
